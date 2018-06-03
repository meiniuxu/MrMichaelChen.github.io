---
layout:     post
title:      PythonTO云歌单
subtitle:   
date:       2018-05-23
author:     CDX
header-img: img/home-bg-art.jpg
catalog: true
tags:
    - Aichen
---
# 前言
最近有一项作业很有意思，软件体系结构需要我们自己实现一个C/S架构的系统，对于没有单独实现过系统的人来说这真是项艰难的任务。
根据自己的经验当然是面向github编程了，呃呃，是这样的。找到了一个模板，就借鉴模板，完成了下载网易云歌单的小程序。
```
import re
import requests
import json
import urllib2
import os

def catchpageinfo(url):
        head_value = { 'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_2) AppleWebKit/537.36\
        (KHTML, like Gecko) Chrome/40.0.2214.115 Safari/537.36'}
        #利用request获得page_text
        page_info = requests.get(url,headers = head_value)
        res = r'<ul class="f-hide">(.*?)</ul>'
        contents = page_info.text
        #print page_info.text
        song_list = re.findall(res, contents, re.S | re.M)
        if (song_list):
        contents = song_list[0]
        else:
        print 'Can not fetch information form URL. Please make sure the URL is right.\n'
        os._exit(0)
        res = r'<li><a .*?>(.*?)</a></li>'
        song_names = re.findall(res, contents, re.S | re.M)
        return song_names

def ensure_download_dir_exist():
        songdir = "song_download_dir"
        if not os.path.exists(songdir):
        os.makedirs(songdir)
        return songdir

def get_songid(value):
        value = value.replace('\\xa0',' ')
        print value
        url = 'http://sug.music.baidu.com/info/suggestion'
        payload = {'word':value,'version':'2','from':'0'}
        
        r = requests.get(url,params=payload)
        contents = r.text
        d = json.loads(contents,encoding="utf-8")
        if('data' not in d):
        return ""
        songid = d["data"]["song"][0]["songid"]
        print "songid:" + songid
        return songid

def get_song_info(songid):
        url = "http://music.baidu.com/data/music/fmlink"
        payload = {'songIds':songid,'type':'flac'}
        r = requests.get(url,params=payload)
        contents = r.text
        d = json.loads(contents,encoding="utf-8")
        return d
        
def extract_song_info(key,dobject):
        name = dobject["data"]["songList"][0][key]
        name = "".join(name.split())
        for c in r'[\\/:*?\"<>|]':
        name = name.replace(c,'')
        return name

def download_song(url):
        minimumsize = 10
        songdir = ensure_download_dir_exist()
        
        #url = re.sub( "#/", "", sys.argv[1] ).strip()
        #print url
        #url = 'https://music.163.com/#/playlist?id=97476391
        song_names = catchpageinfo(url)
        aartistName = ''
        print url
        print song_names
        song_down_list=[]
        for value in song_names:
        songid = get_songid(value)
        if songid == "":
        continue
        
        d = get_song_info(songid)
        if d is not None and 'data' not in d or d['data'] == '' or d['data']["songList"]=='':
        continue
        songlink = d["data"]["songList"][0]["songLink"]
        if(len(songlink) < 10):
        print "\tskip"
        continue
        print "songlink:" + songlink
        
        songName = extract_song_info("songName",d)
        songName = songName.replace('/','%2F').replace('\"',"%22")
        print songName
        
        aartistName = extract_song_info("artistName",d)
        
        filename = "./" + songdir + '/' + songName + "-" + aartistName + ".flac"
        song_down_list.append(songName)
        f = urllib2.urlopen(songlink)
        
        headers = requests.head(songlink).headers
        if 'Content-Length' in headers:
        size = round(int(headers['Content-Length']) / (1024 ** 2),2)
        else:
        continue
        if not os.path.isfile( filename ) or os.path.getsize( filename ) < minimumsize:
        print "%s is downloading to path %s now ......\n" % (songName, filename)
        if size >= minimumsize:
        with open( filename, "wb" ) as code:
        code.write( f.read() )
        print "Download %s is finished.\n" % songName
        else:
        print "the size of %s (%r Mb) is less than 10 Mb, skipping\n\n" % (filename, size)
        else:
        print "%s is already downloaded. Finding next song...\n\n" % songName
        
        print "\n================================================================\n"
        print "Download finish!\nSongs' directory is " + os.getcwd() + "/songs_dir"
        return song_down_list

if __name__ == "__main__":
        download_song("https://music.163.com/playlist?id=2246298053")
```
### 然后就简单的加了个小界面
```
import Tkinter
from catchpage import download_song

class FindLocation( object ):
    def __init__(self):
        # 创建主窗口,用于容纳其它组件
        self.root = Tkinter.Tk()
        #self.root.geometry('500x300+500+200')
        # 给主窗口设置标题内容
        self.root.title( "网易歌单高清下载" )
        # 创建一个输入框,并设置尺寸
        self.url_input = Tkinter.Entry( self.root, width=40 )
        # 创建一个回显列表
        self.display_info = Tkinter.Listbox( self.root, width=50 )
        # 创建一个查询结果的按钮
        self.result_button = Tkinter.Button( self.root, command=self.find_position, text="下载歌曲" )

    # 完成布局
    def gui_arrang(self):
        self.url_input.pack()
        self.display_info.pack()
        self.result_button.pack()

    # 根据url获取歌单歌名
    def find_position(self):
        # 获取输入信息
        self.url_addr = self.url_input.get()
        for item in download_song(self.url_addr):
            self.display_info.insert( 0, item )
        # return download_song(self.url_addr)
        # return song_list
def main():
    # 初始化对象
    FL = FindLocation()
    # 进行布局
    FL.gui_arrang()
    # 主程序执行
    Tkinter.mainloop()
    pass
    
if __name__ == "__main__":
    main()
```
算是个小作业吧，虽然大多还是人家写的。。【捂脸哭】