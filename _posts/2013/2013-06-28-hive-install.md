
<property>  
  <name>hive.metastore.local</name>  
  <value>true</value>  
</property> 
<property>     
<name>javax.jdo.option.ConnectionURL</name>     
<value>jdbc:mysql://10.20.151.10:3306/hive?characterEncoding=UTF-8</value>     
<description>JDBC connect string for a JDBC metastore</description>  
</property> 
<property> 
  <name>javax.jdo.option.ConnectionDriverName</name> 
  <value>com.mysql.jdbc.Driver</value> 
  <description>Driver class name for a JDBC metastore</description> 
</property> 
<property> 
  <name>javax.jdo.option.ConnectionUserName</name> 
  <value>hadoop</value> 
  <description>username to use against metastore database</description> 
</property> 
<property> 
  <name>javax.jdo.option.ConnectionPassword</name> 
  <value>hadoop</value> 
  <description>password to use against metastore database</description> 
</property> 
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://IP:3306/hive?createDatabaseIfNotExist=true&amp;characterEncoding=UTF-8</value>
  <description>JDBC connect string for a JDBC metastore</description>
</property>
 
表或者字段有中文的时候需要修改hive的元数据库的设置。
以mysql为例子，当mysql的字符集设置成utf8的时候使用hive会有问题
(com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Specified key was too long; max key length is 767 bytes ）
 
所以当hive使用mysql作为元数据库的时候mysql的字符集要设置成latin1   default。

为了保存那些utf8的中文，要将mysql中存储注释的那几个字段的字符集单独修改为utf8。
修改字段注释字符集
alter table COLUMNS modify column COMMENT varchar(256) character set utf8;
修改表注释字符集
alter table TABLE_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8;

用mysql做元数据 

alter database hive character set latin1 

这样才不会报  max key length is 767 bytes 