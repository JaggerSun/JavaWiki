DML:处理 
DDL:定义
DCL:控制
1. SELECT COUNT(DISTINCT column1) from table
限制结果集行数
    select * from COLLATIONS limit 10; 用limit 因为mysql不支持top
    select * from COLLATIONS limit 10, 2;跳过前十查两条
    在一个sql中顺便得到总数：
        SELECT SQL_CALC_FOUND_ROWS * FROM collations limit 10;
        SELECT FOUND_ROWS();
        加入了SQL_CALC_FOUND_ROWS 关键字后，就能使用SELECT FOUND_ROWS();来得到总记录数了，比使用count要更加节约资源。
        having， 尽量少用，因为会先查所有再过滤，好处是可以使用统计的比较，一般配合groupby使用
        SELECT character_set_name, 
	GROUP_CONCAT(collation_name ORDER BY id SEPARATOR ', ') FROM collations GROUP BY character_set_name;  把其他不同的字段用分隔符连在一起
    GROUP BY ..WITH ROLLUP加上总数的统计,要配合统计函数使用

2. 修改数据
2.1 备份
    CREATE TABLE newtable SELECT * FROM oldtable;
    INSERT INTO table SELECT * FROM newtable;
    mysqldump
 2.2 插入    
    insert into  AUTO_INCREAMENT 可以插入NULL，会变为自增。
    关联表的时候：   可以用SELECT LAST_INSERT_ID()查出下一个mysql将要分配的值。
    update, delete
3. 创建表，数据库和索引
    CREATE TALBE tablename(
        column1 INT NOT NULL AUTO_INCREMENT,
        column2 INT,
        PRIMARY KEY  (column1),
        KEY index1 (column2),
        CONSTRAINT foreign_key1 FOREIGN KEY () references table2 ()
    )
    ENGINE  = InnoDB
    DEFAULT CHARSET = utf-8 COLLATE = ,,,
    创建索引，  ALTER TABLE table1 ADD INDEX table1(column1)
默认数据修改
        mysql会优化，比如如果数据n<4，vachar会变为char, 如果n>3并且还有其他的vachar,text列的时候会把char改为vachar. 等等。
4 SHOW命令
    SHOW COLUMNS FROM table1;
    SHOW不是标准SQL， mysql提供了一套information_schema虚拟表可以专门用来查询数据库的设计信息：
    select * from information_schema.columns where table_name='collations';