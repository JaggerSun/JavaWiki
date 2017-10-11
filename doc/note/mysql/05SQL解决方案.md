1. 字符串 
1.1 基本函数
      SELECT CONCAT()
        SELECT SBUSTR(column1, int1, int2)
        SELECT IF(CHAR_LENGTH(useId) > 10, CONCAT(LEFT(useId, 7), '...', RIGHT(useId, 5)), useId) FROM test1;  很好的sql，可以在名字过长的时候用
        还有其他的REPLACE, LOCATE等函数
1.2 改变字符集
    SELECT CONVERT(column1 USING utf-8) FROM
    SHOW VARIABLES LIKE 'character_set\_%';
    SET @@sessioncharacter_set_client = 'utf8';
1.3 模板匹配
    _匹配一个\_匹配_ %匹配所有
    正则匹配：不用特别关注
2. 日期和时间
2.1 区间
    BETWEEN .. AND  都是带等号的。
2.2 按年月日进行分组
    SELECT COUNT(*), MONTH(column1) as m FROM table1 GROUP BY m.
    SELECT COUNT(*), LEFT(useId, 5) as m FROM test1 GROUP BY m;
3. ENUM和SET
4. 变量与条件表达式
        变量：
            普通变量@varname  SET @varname=..  能够在其他的地方通过@varname进行引用
            系统和服务变量 @@
            存储过程中的变量
        IF， 上面有用过了
        CASE, END ,两种一种是CASE exp WHEN THEN ELSE 或者是 CASE WHEN exp1 THEN
5. 统计报表
        一些会用到行列倒置 SUM等等

6. 子查询
    IN(SELECT)  ANY(SELECT) EXISTS(SELECT)
    IN可以分解为or, 先把子查询查出M个结果，再进行M此查询
    EXIST 是外表每查出一行的时候都会执行以下EXISTS,返回true的才会计入返回结果
    因此子查询表达用EXISTS， 子查询表小用IN。

7. 


    