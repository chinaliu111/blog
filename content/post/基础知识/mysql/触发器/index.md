---
title: "触发器"
description: 
date: 2024-09-14T16:51:34+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true
categories:
  - 基础知识
tags:
  - mysql
---
在 MySQL 中，触发器（Trigger）是由事件激发并自动执行的存储程序。触发器通常用于在特定的表上进行插入、更新或删除操作时自动执行一些预定义的操作，以维护数据的一致性和完整性。

### 创建触发器

创建触发器时需要指定以下内容：
1. **触发事件**：`INSERT`、`UPDATE` 或 `DELETE`。
2. **触发时机**：`BEFORE` 或 `AFTER`。
3. **目标表**：触发器所作用的表。
4. **触发操作**：触发器激发时执行的 SQL 语句。

### 触发器的基本语法

```sql
CREATE TRIGGER trigger_name
{BEFORE | AFTER} {INSERT | UPDATE | DELETE}
ON table_name
FOR EACH ROW
BEGIN
    -- SQL 语句
END;
```

### 示例

#### 1. 插入触发器
在插入数据到 `orders` 表时，将订单的插入时间记录到 `order_logs` 表中。

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    order_date DATE,
    total_amount DECIMAL(10, 2)
);

CREATE TABLE order_logs (
    log_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    log_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

DELIMITER $$

CREATE TRIGGER after_order_insert
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
    INSERT INTO order_logs (order_id) VALUES (NEW.order_id);
END$$

DELIMITER ;
```

#### 2. 更新触发器
在更新 `employees` 表中的 `salary` 列时，记录旧的和新的薪水值到 `salary_logs` 表中。

```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    salary DECIMAL(10, 2)
);

CREATE TABLE salary_logs (
    log_id INT PRIMARY KEY AUTO_INCREMENT,
    emp_id INT,
    old_salary DECIMAL(10, 2),
    new_salary DECIMAL(10, 2),
    change_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

DELIMITER $$

CREATE TRIGGER before_salary_update
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO salary_logs (emp_id, old_salary, new_salary)
    VALUES (OLD.emp_id, OLD.salary, NEW.salary);
END$$

DELIMITER ;
```

#### 3. 删除触发器
在删除 `products` 表中的记录时，记录删除的产品信息到 `deleted_products` 表中。

```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10, 2)
);

CREATE TABLE deleted_products (
    del_id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT,
    product_name VARCHAR(100),
    deleted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

DELIMITER $$

CREATE TRIGGER after_product_delete
AFTER DELETE ON products
FOR EACH ROW
BEGIN
    INSERT INTO deleted_products (product_id, product_name)
    VALUES (OLD.product_id, OLD.product_name);
END$$

DELIMITER ;
```

### 注意事项

1. **触发器中的 SQL 语句**：触发器内的 SQL 语句不能包含导致触发器再次激发的操作（即不能直接或间接地激发相同的触发器，以防止循环触发）。
2. **触发器的执行顺序**：对于同一事件的多个触发器，执行顺序不保证。
3. **调试和测试**：创建触发器后，建议在开发环境中充分测试其逻辑，以确保在生产环境中不会产生意外结果。

### 删除触发器

```sql
DROP TRIGGER IF EXISTS trigger_name;
```

### 参考文档
- [MySQL 8.0 Reference Manual - Triggers](https://dev.mysql.com/doc/refman/8.0/en/triggers.html)
- [MySQL 5.7 Reference Manual - Triggers](https://dev.mysql.com/doc/refman/5.7/en/triggers.html)

通过学习和使用触发器，你可以实现许多自动化的数据库操作，确保数据的一致性和完整性，并简化数据管理任务。