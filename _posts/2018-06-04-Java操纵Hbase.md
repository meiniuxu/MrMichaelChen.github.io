---
layout:     post
title:      Java操作Hbase
subtitle:   
date:       2018-05-23
author:     CDX
header-img: img/500527815.jpg
catalog: true
tags:
    - Aichen
---
### 今天上了两节课的上机实验，但是一无所获，只是对于甘特图和项目进度的知识有了一点新的了解，阴天，心情很糟。

# Java操作Hbase
凑着课下时间把先前做过的java操作Hbase重新复习了一遍，也加了点自己的东西进去。
## Java连接数据库
```
package hadoop_pro;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.KeyValue;
import org.apache.hadoop.hbase.MasterNotRunningException;
import org.apache.hadoop.hbase.ZooKeeperConnectionException;
import org.apache.hadoop.hbase.client.Delete;
import org.apache.hadoop.hbase.client.Durability;
import org.apache.hadoop.hbase.client.Get;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.client.ResultScanner;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.thrift.generated.Hbase.AsyncProcessor.createTable;
import org.apache.hadoop.hbase.util.Bytes;
import org.apache.hadoop.hive.ql.parse.HiveParser_IdentifiersParser.sysFuncNames_return;
import org.apache.log4j.Logger;

import com.sun.org.apache.xalan.internal.xsltc.compiler.sym;

public class HbaseDao {
	public static Logger log = Logger.getLogger(HbaseDao.class);
	public static Configuration configuration;
	static {
		configuration = HBaseConfiguration.create();
		configuration.set("hbase.zookeeper.quorum", "192.168.1.51");
		//configuration.set("hbase_zookeeper_quorum", "ZooKeeper1,ZooKeeper2");
		configuration.set("hbase.zookeeper.property.clientPort","2181");
		configuration.set("hbase.master", "192.168.1.51:6000");
	}
	public static void main(String[] args) throws IOException {
		System.out.println("Finish！");
	}
	//创建表格
	public static void createTable(String tableName) throws IOException {
		// TODO Auto-generated method stub
		log.info("start create table .....");
		try {
			HBaseAdmin hBaseadmin = new HBaseAdmin(configuration);
			if (hBaseadmin.tableExists(tableName)) {
				// 如果存在数据库 先删除在创建 爱晨
				hBaseadmin.disableTable(tableName);
				hBaseadmin.deleteTable(tableName);
				log.info("------------->"+tableName+" is exist,delete....");
			}
			HTableDescriptor tableDescriptor = new HTableDescriptor(tableName);
			// 添加数据family
			tableDescriptor.addFamily(new HColumnDescriptor("column1"));
			tableDescriptor.addFamily(new HColumnDescriptor("column2"));
			tableDescriptor.addFamily(new HColumnDescriptor("column3"));
			hBaseadmin.createTable(tableDescriptor);
		}catch(MasterNotRunningException e) {
			e.printStackTrace();
		}catch (ZooKeeperConnectionException e) {
			e.printStackTrace();
		}catch (IOException e) {
			e.printStackTrace();
		}
	}
	//插入数据 变量后期定义
	public static void insertData(String rowkey, String tableName,String columnname,String familyname,String keyvalue) {
		try {
			HTable ht = new HTable(configuration,tableName);
			//row key 
			Put put = new Put(rowkey.getBytes());
			put.add(Bytes.toBytes(columnname),familyname.getBytes(),keyvalue.getBytes());
			put.setDurability(Durability.SYNC_WAL);
			ht.put(put);
			ht.close();
		}catch (IOException e) {
			e.printStackTrace();
		}
	}
	// 删除数据表格
	public static void dropTable(String tableName) {
		try {
			HBaseAdmin admin = new HBaseAdmin(configuration);
			admin.disableTable(tableName);
			admin.deleteTable(tableName);
			log.info("------------->"+tableName+"is exist,delete....");
		}catch(MasterNotRunningException e){
			e.printStackTrace();
		}catch(ZooKeeperConnectionException e) {
			e.printStackTrace();
		}catch(IOException e) {
			e.printStackTrace();
		}
	}
	//删除表中的一条记录
	public static void deleteRow(String tablename,String rowkey) {
		// TODO Auto-generated method stub
		try {
			HTable table = new HTable(configuration,tablename);
			List list = new ArrayList();
			Delete d1 = new  Delete(rowkey.getBytes());
			list.add(d1);
			table.delete(list);
			System.out.println("--------->"+rowkey+"删除成功");			
		}catch(IOException e){
			e.printStackTrace();
		}
	}
	//查询所有数据
	public static ResultScanner QueryAll(String tableName) {
		try {
			HTable table = new HTable(configuration,tableName);
			ResultScanner rs = table.getScanner(new Scan());
//			for (Result r : rs) {
//				System.out.println("获得到rowkey:"+new String(r.getRow()));
//				for (KeyValue keyValue : r.raw()) {
//					// System.out.println(new String(keyValue.getFamily()));
//					System.out.println("列"+new String(keyValue.getRow())+"   Family Name:"+new String(keyValue.getQualifier())+"====值："+new String(keyValue.getValue()));
//				}
//			}
			return rs;
		}catch (IOException e) {
			e.printStackTrace();
		}
		return null;
	}
	//单条件查询 根据rowkey查询值
	public static void QueryByCondition1(String rowkey,String tableName) {
		try {
			HTable table = new HTable(configuration,tableName);
			Get scan = new Get(rowkey.getBytes());
			Result r = table.get(scan);
			System.out.println("获得rowkey："+ new String(r.getRow()));
			for (KeyValue keyvalue:r.raw()) {
				System.out.println("列:"+ new String(keyvalue.getFamily()) + "值:" + new String(keyvalue.getValue()));
			}
		}catch (IOException e) {
			e.printStackTrace();
		}
	}
}
```
### 代码有点乱后边有时间我会好好整理一下.
在书写代码的过程中对于Hbase存储数据有了更深的印象，先前还在一直思考Hbase和Hive的区别，但是使用Hive的时候实在太慢，而我又是一些小的简单的数据的处理，所以搞起Hbase。
### 下边的话就是对于上边Dao类的简单引用
```
package hadoop_pro;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.UnsupportedEncodingException;
import java.util.ArrayList;
import java.util.List;
import org.apache.hadoop.hbase.CellUtil;
import org.apache.hadoop.hbase.KeyValue;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.client.ResultScanner;

public class import_data {
	public static void main(String[] args) throws IOException {
		System.out.println("finished!");
	}
	
	public static void show_data(String tablename) throws UnsupportedEncodingException {
		ResultScanner rs = HbaseDao.QueryAll(tablename);
		if(rs != null) {
			for (Result r : rs) {
				System.out.println("获得到rowkey:"+new String(r.getRow()));
				for (KeyValue keyValue : r.raw()) {
					byte[] rb = keyValue.getValueArray();
	                 String row = new String(r.getRow(),"UTF-8");
	                 String family = new String(CellUtil.cloneFamily(keyValue),"UTF-8");
	                 String qualifier = new String(CellUtil.cloneQualifier(keyValue),"UTF-8");
	                 String value = new String(CellUtil.cloneValue(keyValue));
	                 System.out.println("[row:"+row+"],[family:"+family+"],[qualifier:"+qualifier+"],[value:"+value+"]");
				}
			}
		}
	}
	private static void read_data(String fileName) {
		// TODO Auto-generated method stub
		File file = new File(fileName);
		InputStream in = null;
		try {
			in = new FileInputStream(file);
			int tempbyte;
			while((tempbyte = in.read()) != -1) {
				System.out.println(tempbyte);
			}
			in.close();
		}catch (IOException e) {
			e.printStackTrace();
			return;
		}	
	}
	public static List readFileByLines(String fileName) {
		File file = new File(fileName);
		BufferedReader reader = null;
		List list = new ArrayList();
		try {
			reader = new BufferedReader(new FileReader(file));
			String tempString = null;
			int line = 1;
			String[] resultbuffer;
			while((tempString = reader.readLine()) != null) {
				System.out.println(tempString);
				String[] bufferlist;
				bufferlist = tempString.split(" ");
				//返回按照空格分割好的列表
				for (String buff : bufferlist) {
					list.add(buff);
					//System.out.println(buff);
				}
			}
		}catch(IOException e){
			e.printStackTrace();
		}finally {
			if (reader != null) {
				try {
					reader.close();
				}catch(IOException el) {
					el.printStackTrace();
				}
			}
		}
		//System.out.println(list.toString());
		return list;
	}
	private static void getFile(String path){   
	        // get file list where the path has   
	        File file = new File(path);   
	        // get the folder list   
	        File[] array = file.listFiles();   
	          
	        for(int i=0;i<array.length;i++){   
	            if(array[i].isFile()){   
	                // only take file name   
	                System.out.println("^^^^^" + array[i].getName());   
	                // take file path and name   
	                System.out.println("#####" + array[i]);   
	                // take file path and name   
	                System.out.println("*****" + array[i].getPath());   
	            }else if(array[i].isDirectory()){   
	                getFile(array[i].getPath());   
	            }   
	        }   
	    }   
}   
```
完