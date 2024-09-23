---
title: "MySQL中快速创建大量测试数据"
date: 2019-02-08T13:57:45+08:00
lastmod: 2019-02-08T13:57:45+08:00
draft: false
keywords: ["mysql","大量","测试数据"]
description: "简要介绍如何在MySQL中快速创建大量测试数据以方便进行性能相关的测试与研究"
tags: ["mysql"]
categories: ["数据库"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: true
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: true

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""
---

在学习`MySQL`索引和分库分表等知识的过程中，经常会涉及到创建大批量的测试数据，本文简要说明自己常用的几种创建方式以及各自的优劣对比。

<!--more-->

## 实现方式

以下述的`system_user`表为例分别说明在不同的方式下如何大批量的创建测试数据。

```sql
CREATE TABLE `system_user` (
	`id` INT NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(8) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
	`age` INT DEFAULT NULL,
	`tag` VARCHAR(8) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL,
	PRIMARY KEY (`id`)
) ENGINE = INNODB CHARSET = utf8;
```

### 通过代码程序创建

工作中使用的编程语言主要是`Java`，在之前我不熟悉`MySQL`存储过程用法的时，主要采用[**JDBC**](https://docs.oracle.com/javase/tutorial/jdbc/basics/index.html)的方式实现批量创建数据，代码类似如下:

```java
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.RandomStringUtils;
import org.apache.commons.lang3.RandomUtils;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import javax.annotation.Resource;
import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

@Slf4j
@SpringBootTest
public class TestBatchInsertData {

    @Resource
    private DataSource dataSource;

    private int DATA_SIZE = 1000_0000;
    private int BATCH_SIZE = 100;

    @Test
    public void testAddBatchData() {
        String insertSQL = "insert into system_user(name,age,tag) VALUES(?,?,?)";
        try (Connection conn = dataSource.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(insertSQL)) {
            conn.setAutoCommit(false);
            for (int i = 1; i <= DATA_SIZE; i++) {
                pstmt.setString(1, RandomStringUtils.randomAlphanumeric(8));
                pstmt.setInt(2, RandomUtils.nextInt(18, 80));
                pstmt.setString(3, RandomStringUtils.randomAlphabetic(8));
                pstmt.addBatch();
                if (i % BATCH_SIZE == 0) {
                    pstmt.executeBatch();
                    conn.commit();
                    log.info("执行一次批量提交:\t" + i / BATCH_SIZE);
                }
            }
            pstmt.executeBatch();
            conn.commit();
            log.info("完成数据批量插入");
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
}
```

从上述代码可知，此种实现方式较为简洁，实际的业务代码只有20行左右，对于具有`Java`开发经验的人来说上手很快，不足之处是需要额外准备相应的执行环境。

### 通过存储过程创建

```sql
DELIMITER $$

USE `test`$$

DROP PROCEDURE IF EXISTS `add_user_batch`$$

CREATE DEFINER=`root`@`%` PROCEDURE `add_user_batch`(IN COUNT INT)
BEGIN
    DECLARE i INT;
    DECLARE t_name VARCHAR(8);
    DECLARE t_tag VARCHAR(20);
    DECLARE t_age INT(2);
    DECLARE t_sql_template VARCHAR(100);
    DECLARE t_sql TEXT;   
    DECLARE t_tag_mod_val INT DEFAULT(25);
    DECLARE t_commit_mod_val INT DEFAULT(100);
    
    DECLARE t_start_time DATETIME;
    DECLARE t_end_time DATETIME;    
    
    TRUNCATE TABLE `system_user`;
    
    SET t_start_time=NOW();
    SET t_sql_template = "INSERT INTO `system_user`(NAME, age, tag) VALUES";
    SET t_sql = t_sql_template;
    SET i = 1;
    WHILE i <= COUNT
        DO
            SET t_age = FLOOR(1 + RAND() * 60);
            SET t_name = LEFT(UUID(), 8);
            -- 给tag随机制造空值
            IF MOD(i, t_tag_mod_val) = 0 THEN
                SET t_tag = "NULL";
            ELSE
                SET t_tag = CONCAT("'",LEFT(UUID(), 8),"'");
            END IF;
 
            SET t_sql = CONCAT(t_sql,"('",t_name,"',",t_age,",",t_tag,")");
            
            IF MOD(i,t_commit_mod_val) != 0 THEN
              SET t_sql = CONCAT(t_sql,",");
            ELSE
              SET t_sql = CONCAT(t_sql,";");
                   -- 只要达到t_commit_mod_val要求的次数，就执行并提交
                   SET @insert_sql = t_sql;
                   PREPARE stmt FROM @insert_sql;
                   EXECUTE stmt;
                   DEALLOCATE PREPARE stmt;
                   COMMIT;
              SET t_sql=t_sql_template;
            END IF;
            SET i = i + 1;
        END WHILE;
        
        -- 不能被t_commit_mod_val整除时，余下的数据处理
        IF LENGTH(t_sql) > LENGTH(t_sql_template) THEN
                   SET t_sql=CONCAT(SUBSTRING(t_sql,1,LENGTH(t_sql)-1),';');
                   SET @insert_sql = t_sql;
                   PREPARE stmt FROM @insert_sql;
                   EXECUTE stmt;
                   DEALLOCATE PREPARE stmt;
                   COMMIT;
        END IF;
        SET t_end_time=NOW();
        SELECT CONCAT('insert data success,time cost ',TIMEDIFF(t_end_time,t_start_time)) AS finishedTag;
END$$

DELIMITER ;
```

调用过程类似如下:

```sql
-- 清空原有记录
TRUNCATE TABLE `system_user`;

-- 调用存储过程
CALL add_user_batch(1000);

-- 验证查询结果
SELECT COUNT(*) FROM `system_user`;
```

可以看出虽然用存储过程也能实现批量提交，但相对于`Java`实现而言，其代码量更大更为复杂，上手门槛略高。不过其好处也很明显，只要有数据库环境就能执行[^1]。

### 通过SQL语句创建

此种方式需要通过[**SELECT INSERT INTO**](https://www.w3schools.com/sql/sql_insert_into_select.asp)来实现，具体步骤如下：

1. 通过如下的SQL初始化表中的数据：

   ```sql
   INSERT INTO `system_user`(NAME,age,tag) VALUES
   (LEFT(UUID(), 8),FLOOR(1 + RAND() * 60),LEFT(UUID(), 8));
   ```

2. 根据实际需要多次执行下述SQL：

   ```sql
   INSERT INTO `system_user`(NAME,age,tag) 
   SELECT LEFT(UUID(), 8),FLOOR(1 + RAND() * 60),LEFT(UUID(), 8) FROM `system_user`;
   ```

这种方式主要是利用了`SELECT`每次查询之前表中的全部数据，然后重新插入，每次`SELECT`时都会将表中已有的数据全部查询出来，获取出总的记录数，然后根据`SELECT`后面的条件重新对每一条插入的记录重新赋值插入，实际上相当于翻倍插入。

由于每次执行`SELECT INTO`时都是将之前的数据量扩大1倍，故往数据库中插入的总数`count`与执行次数`n`的关系如下:

> **count = $2^n$**

更具体的信息如下：

| 阶段       | 插入总数 |
| ---------- | -------- |
| 第1次执行  | 2        |
| 第2次执行  | 4        |
| 第3次执行  | 8        |
| ...        |          |
| 第10次执行 | 1024     |
| ...        |          |
| 第20次执行 | 1048576  |
| 第21次执行 | 2097152  |
| 第22次执行 | 4194304  |
| 第23次执行 | 8388608  |

在`MySQL`中`tmp_table_size`为16M，`innodb_buffer_pool_size`的默认值是128M，当执行到一定次数后，会出现类似`The total number of locks exceeds the lock table size`的错误，此时需要根据实际情况调整这两个参数，参考如下：

```sql
SET GLOBAL tmp_table_size =512*1024*1024; -- 512M
SET global innodb_buffer_pool_size= 2*1024*1024*1024; -- 2G
```

此种方式虽然需要多次执行SQL语句，但其优点也很明显，只需要将SQL语句稍作修改，就能适用于不同的数据库表。

## 对比&总结

各种方式的对比如下：

|                  | 优点                                                         | 缺点                                                         | 适用场景                                             |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------------- |
| **代码程序创建** | 1.实现方便，有编程知识即可<br>2.性能不受数据量大小的影响     | 1.需要具备相关的编程知识<br>2.需要有专门的软件运行环境，移植性不好<br>3.不具备通用性 | 需要反复的创建大批量数据                             |
| **存储过程创建** | 1.由于直接操作数据库，`速度最快`<br>2.编写完成后可重复使用<br>3.性能不受数据量大小的影响 | 1.存储过程的编写耗时，调试不太方便<br>2.不具备通用性         | 1.需要反复的创建大批量数据<br>2.无法通过程序代码创建 |
| **SQL语句创建**  | 1.使用最方便，只要有mysql环境就能上手<br>2.`具备通用性`，适合各种数据库表 | 1.需要多次执行SQL语句<br>2.越到后面单次数据量越大，影响性能  | 创建大批量测试数据的次数较少,通常1到2次              |

各自执行1000万条数据的粗略耗时对比如下：[^2]

* **代码程序创建**，6小时
* **存储过程创建**，11分钟
* **SQL语句创建**，5分钟

从上述结果可以看出通过**SQL语句创建**耗时相对较少[^3]，若只是单纯的插入数据，建议优先选择此种方式。

{{% admonition note "说明" false %}}

由于**代码程序创建**和**存储过程创建**这2种方式与特定的数据表强相关，换做其它表后需要重新修改，故在没有特殊要求的情况下建议采用**SQL语句创建**。

{{% /admonition %}}



{{% admonition question "疑问" false %}}

按**存储过程创建**从理论上来说速度要比**SQL语句创建**快很多(存储过程可以控制每次提交的数据量大小)，但多次测试发现前者的耗时约为后者的1倍，原因待继续分析。

{{% /admonition %}}



[^1]: 前提是有创建存储过程的权限
[^2]: 基于相同的提交次数进行比较，且是在个人电脑环境，只做横向对比，不具有很强的参考价值
[^3]:  代码程序创建和存储过程创建中涉及到一些控制逻辑和随机数的生成，这些是造成其耗费时间较长的因素之一