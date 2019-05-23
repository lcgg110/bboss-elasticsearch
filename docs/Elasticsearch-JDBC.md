# Elasticsearch JDBC案例介绍

The best elasticsearch highlevel java rest api-----bboss     

Elasticsearch  6.3以后的版本可以通过jdbc操作es，该功能还在不断的完善当中，本文介绍es jdbc使用方法。

# 1.首先在工程中导入es jdbc maven坐标：

```xml
导入elasticsearch jdbc驱动和bboss持久层
<dependency>
  <groupId>org.elasticsearch.plugin</groupId>
  <artifactId>jdbc</artifactId>
  <version>6.3.2</version>
</dependency>
<dependency> 
    <groupId>com.bbossgroups</groupId> 
    <artifactId>bboss-persistent</artifactId> 
    <version>5.3.6</version> 
</dependency> 

在pom中添加elastic maven库 

<repositories>
  <repository>
    <id>elastic.co</id>
    <url>https://artifacts.elastic.co/maven</url>
  </repository>
</repositories>
```





# 2.通过jdbc驱动执行elasticsearch sql相关功能

- 启动es数据源
- 执行elasticsearch sql相关功能

直接看执行各种sql功能的代码[ESJdbcTest](https://gitee.com/bboss/bestpractice/blob/master/persistent/src/com/frameworkset/sqlexecutor/ESJdbcTest.java)：

```
package com.frameworkset.sqlexecutor;
/*
 *  Copyright 2008 biaoping.yin
 *
 *  Licensed under the Apache License, Version 2.0 (the "License");
 *  you may not use this file except in compliance with the License.
 *  You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 *  Unless required by applicable law or agreed to in writing, software
 *  distributed under the License is distributed on an "AS IS" BASIS,
 *  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 *  See the License for the specific language governing permissions and
 *  limitations under the License.
 */

import com.frameworkset.common.poolman.SQLExecutor;
import com.frameworkset.common.poolman.util.SQLUtil;
import org.junit.Test;

import java.sql.SQLException;
import java.util.HashMap;
import java.util.List;

public class ESJdbcTest {

	public void initDBSource(){
		SQLUtil.startPool("es",//ES数据源名称
				"org.elasticsearch.xpack.sql.jdbc.jdbc.JdbcDriver",//ES jdbc驱动
				"jdbc:es://http://127.0.0.1:9200/timezone=UTC&page.size=250",//es链接串
				"elastic","changeme",//es x-pack账号和口令
				"SHOW tables 'dbclob%'" //数据源连接校验sql
		);
	}

	/**
	 * 执行一个查询
	 * @throws SQLException
	 */
	@Test
	public void testSelect() throws SQLException {
		initDBSource();//启动数据源
		//执行查询，将结果映射为HashMap集合
		 List<HashMap> data =	SQLExecutor.queryListWithDBName(HashMap.class,"es","SELECT SCORE() as score,content as content FROM dbclobdemo");
		 System.out.println(data);
	}

	/**
	 * 进行模糊搜索，Elasticsearch 的搜索能力大家都知道，强！在 SQL 里面，可以用 match 关键字来写，如下：
	 * @throws SQLException
	 */
	@Test
	public void testMatchQuery() throws SQLException {
		initDBSource();
		List<HashMap> data =	SQLExecutor.queryListWithDBName(HashMap.class,"es","SELECT SCORE(), * FROM dbclobdemo WHERE match(content, '_ewebeditor_pa_src') ORDER BY documentId DESC");
		System.out.println(data);

		/**
		 *还能试试 SELECT 里面的一些其他操作，如过滤，别名，如下：
		 */
		data =	SQLExecutor.queryListWithDBName(HashMap.class,"es","SELECT SCORE() as score,title as myname FROM dbclobdemo  as mytable WHERE match(content, '_ewebeditor_pa_src') and (title.keyword = 'adsf' OR title.keyword ='elastic') limit 5 ");
		System.out.println(data);
	}
	/**
	 * 分组和函数计算
	 */
	@Test
	public void testGroupQuery() throws SQLException {
		initDBSource();
		List<HashMap> data =	SQLExecutor.queryListWithDBName(HashMap.class,"es","SELECT title.keyword,max(documentId) as max_id FROM dbclobdemo as mytable group by title.keyword limit 5");
		System.out.println(data);


	}


	/**
	 * 查看所有的索引表
	 * @throws SQLException
	 */
	@Test
	public void testShowTable() throws SQLException {
		initDBSource();
		List<HashMap> data =	SQLExecutor.queryListWithDBName(HashMap.class,"es","SHOW tables");
		System.out.println(data);
	}

	/**
	 * 如 dbclob 开头的索引，注意通配符只支持 %和 _，分别表示多个和单个字符（什么，不记得了，回去翻数据库的书去！）
	 * @throws SQLException
	 */
	@Test
	public void testShowTablePattern() throws SQLException {
		initDBSource();
		List<HashMap> data =	SQLExecutor.queryListWithDBName(HashMap.class,"es","SHOW tables 'dbclob_'");
		System.out.println(data);
		data =	SQLExecutor.queryListWithDBName(HashMap.class,"es","SHOW tables 'dbclob%'");
		System.out.println(data);
	}
	/**
	 * 查看索引的字段和元数据
	 * @throws SQLException
	 */
	@Test
	public void testDescTable() throws SQLException {
		initDBSource();
		List<HashMap> data =	SQLExecutor.queryListWithDBName(HashMap.class,"es","DESC dbclobdemo");
		System.out.println(data);
		data =	SQLExecutor.queryListWithDBName(HashMap.class,"es","SHOW COLUMNS IN dbclobdemo");
		System.out.println(data);
	}

	/**
	 * 不记得 ES 支持哪些函数，只需要执行下面的命令，即可得到完整列表
	 * @throws SQLException
	 */
	@Test
	public void testShowFunctin() throws SQLException {
		initDBSource();
		List<HashMap> data =	SQLExecutor.queryListWithDBName(HashMap.class,"es","SHOW FUNCTIONS");
		System.out.println(data);
		//同样支持通配符进行过滤：
		data =	SQLExecutor.queryListWithDBName(HashMap.class,"es","SHOW FUNCTIONS 'S__'");
		System.out.println(data);

	}
}
```

如果执行的时候报错：

![img](https://oscimg.oschina.net/oscnet/b84c48aeb829871a1b394a42027b8b1a52c.jpg)

可以采用正式的license或者在elasticsearch.yml文件最后添加以下配置即可：

**xpack.license.self_generated.type: trial**





# 3.bboss 针对es jdbc的替代解决方案

 

bboss 提供一组sql和fetchQuery API，可替代es jdbc模块；采用bboss即可拥有bboss的客户端自动发现和容灾能力、对es、jdk、spring boot的兼容性能力，又可以拥有es jdbc的所有功能，同时还解决了因为引入es jdbc导致项目对es版本的强依赖和兼容性问题，参考demo：

orm查询
<https://gitee.com/bbossgroups/eshelloword-booter/blob/master/src/test/java/org/bboss/elasticsearchtest/sql/SQLOrmTest.java>
分页查询
<https://gitee.com/bbossgroups/eshelloword-booter/blob/master/src/test/java/org/bboss/elasticsearchtest/sql/SQLPagineTest.java>





# 开发交流

elasticsearch sql官方文档：

<https://www.elastic.co/guide/en/elasticsearch/reference/current/xpack-sql.html>

bboss elasticsearch交流：166471282

**bboss elasticsearch微信公众号：**

<img src="https://static.oschina.net/uploads/space/2017/0617/094201_QhWs_94045.jpg"  height="200" width="200">



# 支持我们

<div align="left"></div>

<img src="E:/workspace/bbossgroups/bboss-elastic/docs/images/alipay.png"  height="200" width="200">
