# 从零起步学习MySQL || 第二章：DDL语句定义及常见用法示例
<img width="1145" height="799" alt="image" src="https://github.com/user-attachments/assets/e8f77779-83bc-4141-8ca8-22e3cfbe8bed" />

## 一、什么是 DDL？
DDL（Data Definition Language）是 SQL 的一个分类，用于定义和管理数据库对象结构。
执行 DDL 语句时，MySQL 会自动提交事务（自动生效），不可回滚。

常见的 DDL 操作包括：

| 操作 | 关键字 | 功能说明 |
| ---- | ---- | ---- |
| 创建 | `CREATE` | 创建数据库、表、视图、索引等 |
| 修改 | `ALTER` | 修改数据库或表的结构 |
| 删除 | `DROP` | 删除数据库、表、视图、索引等 |
| 重命名 | `RENAME` | 修改数据库对象的名称 |
| 清空 | `TRUNCATE` | 清空表中的数据（但不删除表结构） |

## 二、DDL 的常见语句与示例
### 1. 创建数据库：`CREATE DATABASE`
```sql
CREATE DATABASE IF NOT EXISTS mydb
CHARACTER SET utf8mb4
COLLATE utf8mb4_general_ci;
```
**解释**：
- `IF NOT EXISTS`：防止重复创建；
- `CHARACTER SET`：指定字符集；
- `COLLATE`：指定排序规则。

**查看数据库**：
```sql
SHOW DATABASES;
```

**使用数据库**：
```sql
USE mydb;
```

### 2. 创建数据表：`CREATE TABLE`
```sql
CREATE TABLE user (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(100) NOT NULL,
    gender ENUM('男', '女') DEFAULT '男',
    age INT CHECK(age >= 0),
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
**说明**：
- `id`：主键（`PRIMARY KEY`），自动增长；
- `username`：不允许为空（`NOT NULL`），唯一（`UNIQUE`）；
- `gender`：枚举类型（`ENUM`）；
- `CHECK`：字段约束；
- `create_time`：默认当前时间戳。

**查看表结构**：
```sql
DESC user;
```

### 3. 修改表结构：`ALTER TABLE`
#### （1）添加列
```sql
ALTER TABLE user ADD email VARCHAR(100);
```

#### （2）修改列类型
```sql
ALTER TABLE user MODIFY age SMALLINT;
```

#### （3）重命名列
```sql
ALTER TABLE user CHANGE username user_name VARCHAR(50);
```

#### （4）删除列
```sql
ALTER TABLE user DROP COLUMN gender;
```

#### （5）添加约束
```sql
ALTER TABLE user ADD CONSTRAINT uq_email UNIQUE(email);
```

#### （6）修改表名
```sql
ALTER TABLE user RENAME TO users;
```

### 4. 删除表：`DROP TABLE`
```sql
DROP TABLE IF EXISTS users;
```
**注意**：
- 删除后，数据和结构都会被永久删除；
- `IF EXISTS` 可以避免错误。

### 5. 清空表数据：`TRUNCATE TABLE`
```sql
TRUNCATE TABLE user;
```

**区别于 `DELETE`**：

| 特点 | `DELETE` | `TRUNCATE` |
| ---- | ---- | ---- |
| 是否为 DDL | 否（DML） | 是（DDL） |
| 是否可回滚 | ✅ 可回滚（事务内） | ❌ 不可回滚 |
| 是否重置自增 | ❌ 否 | ✅ 是 |
| 执行速度 | 较慢 | 非常快 |

## 三、DDL 的执行特性
1. DDL 语句会自动提交事务；
2. 一旦执行，无法通过 `ROLLBACK` 回滚；
3. 一般用于数据库初始化、表结构变更阶段；
4. 在 Java 项目中通常通过数据库迁移工具（如 Flyway、Liquibase）管理。

## 四、综合示例
创建一个电商系统的 `product` 表：
```sql
CREATE TABLE product (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL CHECK (price >= 0),
    stock INT DEFAULT 0,
    category_id INT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES category(id)
);
```
**要点说明**：
- `DECIMAL(10,2)`：精确存储价格；
- `ON UPDATE CURRENT_TIMESTAMP`：每次修改记录时自动更新时间；
- `FOREIGN KEY`：建立外键约束，保证表间关联。

## 五、在 Java 中如何使用 DDL（简单示例）
在 Java 项目中，你可以使用 JDBC 执行 DDL：
```java
String sql = """
    CREATE TABLE IF NOT EXISTS user (
        id INT PRIMARY KEY AUTO_INCREMENT,
        username VARCHAR(50) NOT NULL,
        password VARCHAR(100) NOT NULL
    )
""";

try (Connection conn = DriverManager.getConnection(url, user, password);
     Statement stmt = conn.createStatement()) {
    stmt.executeUpdate(sql);
    System.out.println("表创建成功！");
}
```

## ✅ 总结

| 操作 | 语句 | 功能 |
| ---- | ---- | ---- |
| 创建 | `CREATE DATABASE / TABLE` | 定义数据库或表 |
| 修改 | `ALTER TABLE` | 修改表结构 |
| 删除 | `DROP DATABASE / TABLE` | 删除数据库或表 |
| 清空 | `TRUNCATE TABLE` | 清空表数据但保留结构 |
| 重命名 | `RENAME TABLE` | 修改表名 |
