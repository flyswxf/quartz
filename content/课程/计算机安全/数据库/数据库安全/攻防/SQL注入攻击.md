# SQL注入攻击 (SQL Injection)

SQL 注入攻击是应用层最常见的安全漏洞之一。攻击者通过在应用程序的输入字段中插入恶意 SQL 语句，欺骗后端数据库执行非预期的操作。

## 1. 攻击原理
当应用程序直接将用户输入拼接到 SQL 查询字符串中，而没有进行充分的过滤或转义时，攻击者就可以通过闭合原有的 SQL 语句并附加恶意代码，从而改变查询的逻辑。

## 2. 常见攻击类型与实例

### 2.1 认证绕过 (Authentication Bypass)
#### 案例1
假设验证用户登录的 SQL 语句如下：
```sql
SELECT * FROM users WHERE username = '输入用户名' AND password = '输入密码';
```
如果用户在用户名处输入 `admin' OR '1'='1`，密码处随意输入，拼接后的 SQL 变成：
```sql
SELECT * FROM users WHERE username = 'admin' OR '1'='1' AND password = '...';
```
由于 `'1'='1'` 永远为真，导致查询返回所有记录或特定记录，从而成功绕过身份验证。

#### 案例2
假设应用程序使用如下方式拼接 SQL：
```sql
SELECT * FROM users WHERE username = ' $user_input ' AND password = ' $password_input ';
```
如果攻击者在用户名中输入 `admin' --`，拼接后的语句变为：
```sql
SELECT * FROM users WHERE username = 'admin' -- ' AND password = '...';
```
`--` 是 SQL 的注释符，导致密码验证部分被忽略，攻击者直接以 admin 身份登录。

### 2.2 联合查询注入 (UNION-Based SQLi)
攻击者利用 `UNION` 操作符将恶意查询的结果附加到合法查询的结果集后，从而窃取数据库中的敏感数据。
```sql
SELECT title, content FROM news WHERE id = 1 UNION SELECT username, password FROM users;
```

### 2.3 盲注 (Blind SQLi)
当应用程序不直接返回数据库错误信息或查询结果时，攻击者通过构造特定 SQL 语句，观察页面的响应差异（布尔盲注）或响应时间延迟（时间盲注）来逐个字符地推测数据库内容。

## 3. 防御机制

- **参数化查询 / 预编译语句 (Parameterized Queries / Prepared Statements)**：这是防御 SQL 注入最有效的方法。数据库将 SQL 逻辑与数据参数严格分离，即使参数中包含 SQL 关键字，也不会被解析为可执行指令。
- **输入验证与过滤 (Input Validation)**：对所有用户输入进行白名单验证，确保数据类型、长度和格式合法。
- **最小权限原则 (Principle of Least Privilege)**：应用程序连接数据库的账户应仅拥有完成特定任务所需的最低权限，禁止使用 `sa` 或 `root` 账户。
- **使用 ORM 框架**：现代对象关系映射（ORM）框架通常会自动处理参数转义，减少直接拼接 SQL 的风险。
