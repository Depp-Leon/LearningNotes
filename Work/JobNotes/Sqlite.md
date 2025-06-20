#####  SQlite

1. sqlite的常用命令

   ```
   sqlite3 <数据库.db>
   .tables   #查看所有表
   SELECT * FROM TABLE;    #类似于sql语句,注意需要加分号
   .quit 
   .exit	#退出
   
   ```

2. 项目中sqlite的路径

   ```
   /opt/apps/chenxinsd/cache/xxx.db
   ```

3. Sqlite3查看表的结构

   ```
   PRAGMA table_info(table_name);			#在Sqlite下
   ```

   ```
   #表的结构如下
   cid   name        type       notnull   dflt_value   pk
   0     id          INTEGER    1         NULL         1
   1     name        TEXT       0         NULL         0
   2     age         INTEGER    0         NULL         0
   cid: 字段的序号（从 0 开始）。
   name: 字段的名称。
   type: 字段的数据类型。
   notnull: 是否允许 NULL 值（1 表示不允许）。
   dflt_value: 默认值。
   pk: 是否为主键（1 表示是主键）
   ```

4. C++提取Sqlite3数据库的两种方式

   ```c++
   #方式一：使用sqlite3_get_table函数
   
   /**
    * @brief: 读取sqlite表中数据
    * @param db: 打开的数据对象
    * @param zSql: 命令
    * @param pazResult: 读到的数据
    * @param pnRow: 行数
    * @param pnColumn: 列数
    * @retval: sqlite3_get_table的返回值
    */
   int jy_sqlite_get_table(sqlite3 *db, const char *zSql, char ***pazResult, int *pnRow, int *pnColumn)
   {
       int res;
       char *err_msg;
       res = sqlite3_get_table(db, zSql, pazResult, pnRow, pnColumn, &err_msg);
       if (res != SQLITE_OK)
           LOG(ERROR) << "CDBTimedTasksOper Select! err:[" << err_msg << "]";
       if (err_msg)
           sqlite3_free(err_msg);
       return res;
   }
   ```

   ```c++
   #方式二：使用sqlite3_exec函数
   
   bool CTerminalDetailsCache::setLastScanInfo(const std::string &result, const std::string &time)
   {
       if (!m_sqlite) {
           return false;
       }
       char *errmsg = nullptr;
       char sql[1024] = {0};
       snprintf(sql, sizeof(sql), "UPDATE terminal_details SET last_scan_result = \"%s\", last_scan_time = \"%s\" WHERE id = 1;", 
                result.c_str(), time.c_str());
       int rc = sqlite3_exec(m_sqlite, sql, nullptr, nullptr, &errmsg);
       if (rc != SQLITE_OK) {
           sqlite3_free(errmsg);
           return false;
       }
       return true;
   }
   ```

5. sqlite3的存储方式：任何数据都是以字符方式存储的。

   1. SQLite数据以文本形式（TXET,SQLITE原生自带的数据类型之一）存储
   2. SQLite 列的声明类型（如 `INTEGER`、`TEXT`、`REAL`）只是一个“建议”，而不是严格限制。
   3. SQLite3 中，即使存储的数据是整数、浮点数或其他类型，**查询结果返回时，默认是以字符形式（文本）返回的**。数据以一种通用的动态类型系统存储，取出时通常是字符形式

6. 从sqlite3中取出字符串类型的时间戳。可以使用`long`长整型给`time_t`赋值

   1. `int64_t`规定了64位的整数。适合跨平台。

   2. `long`在不同的系统可能是32位或者64位。

   3. 在现代的 64 位 Linux 系统上，`time_t` 通常是 64 位的（`long` 类型）。所以它俩可以给时间戳赋值

   ```
   time_t addTime =  std::strtol(p_result[i*col], nullptr, 10);
   #使用 std::strtol 将字符串 p_result[i * col] 转换为一个 long 整数，并赋值给 info.addTime。
   1. p_result[i * col] 是一个 char* 指针，指向字符串数据。
   2. nullptr 表示不需要返回剩余未解析的字符串部分。
   3. 10 是基数，表示字符串是十进制格式
   
   #错误方式：
   time_t addTime = *p_result[i * col];
   直接用*取地址，得到的是第一个字符的值。字符指针不能直接取值
   
   #补充：std::atoi() 将字符串转为int类型
   int typeValue =  std::atoi(p_result[i*col+2]);
   ```

7. SQLite 接受 `CHAR(n)` 的语法是为了兼容其他数据库，但不会严格执行固定长度。如果你在 SQLite 中定义了一个列为 `CHAR(260)`，SQLite 会将其解析为具有 `TEXT` 亲和性的列，忽略长度限制（260）。

   1. **SQlite会根据类型亲和性和数据内容选择最合适的内部表示形式**；你可以在建表时为列指定类型（如 INTEGER、TEXT、REAL），但这只是**类型亲和性（Type Affinity）**，而**不是强制类型约束**。SQLite 允许任何列存储任何类型的数据（整数、文本、浮点数等），但会**根据列的类型亲和性和插入的数据类型决定如何存储**

   2. SQLite 的原生数据类型（类型亲和性）：

      包含 INT → INTEGER亲和性。

      包含 CHAR, CLOB, TEXT → TEXT亲和性。

      包含 BLOB → BLOB亲和性。

      包含 REAL, FLOA, DOUB → REAL亲和性。

      其他（如 DECIMAL,、DATE、TIMESTAMP）→ **NUMERIC**亲和性。

      无类型声明 → 无亲和性（接受任何数据）。

      > 即建表时指定了类型只是为了迎合其他数据库的类型，实际会转会为原生类型。如果转换成功，数据会以转换后的格式存储；如果转换失败，SQLite 会保留输入值的原始形式。而**不会强制报错**

   3. **NUMERIC 亲和性**意味着

      - 如果插入的数据是整数或浮点数，会尽量存储为数值。
      - 如果插入的是字符串，会尝试将其转换为数值（如果无法转换，则按字符串存储）。

   4. **SQLite 的存储是动态的，具体存储什么取决于你插入的数据，而不是列声明的类型**；动态类型的实际效果

      - 插入时
        - SQLite 会根据列的亲和性和输入值决定存储格式。
        - 例如，INTEGER 列插入 "123" 会转为整数，但插入 "abc" 保持文本。
      - 查询时
        - 数据以存储时的格式返回，API 调用（如 sqlite3_column_int）会尝试转换。
        - 例如，从 TEXT 列取值时，sqlite3_column_int 会尝试将文本转为整数。

8. sqlite3复合主键

   ```
   primary key(name, type)
   ```

   这是一个**复合主键**（Composite Primary Key），意味着表中每一行的 (name, type) 组合必须是唯一的；

   单独的 name 或 type 值可以重复，但它们的组合不能重复。
   
9. sqlite3在调用open打开数据库的时候，如果不存在该数据库将会创建一个新的

   ```
   std::string path;
   sqlite3* sourceDb;
   (sqlite3_open(path.c_str(), &sourceDb)
   ```

10. C++中使用sqlite3的一些语法

    1. 打开数据库

       ```c++
       sqlite3 *db = nullptr;
       int rc = sqlite3_open("test.db", &db);		//test.db是路径
       
       // 打开数据库(只读)
       int rc = sqlite3_open_v2(sqlSourcePath.c_str(), &sqlSource, SQLITE_OPEN_READONLY, nullptr);
       // 打开目标数据库（读写）
       rc = sqlite3_open_v2(sqlDestPath.c_str(), &sqlDest, SQLITE_OPEN_READWRITE | SQLITE_OPEN_CREATE, nullptr);
       
       // sqlite3_open_v2 和 sqlite3_open的区别在于sqlite3_open_v2更灵活，可以指定打开模式（只读/只写/创建等）
       ```

       > **作用**：打开（或创建）一个数据库文件，返回数据库连接句柄 `db`。

    2. 预编译 SQL 语句

       ```c++
       sqlite3_stmt *stmt = nullptr;
       const char *sql = "INSERT INTO users (name, age) VALUES (?, ?)";
       int rc = sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr);
       
       // sqlite3_stmt 是SQLite 预编译语句对象的类型，它代表一条已经编译好的 SQL 语句，可以多次绑定参数、执行、重置，效率高
       ```

       > **作用**：将 SQL 语句编译成可执行的预编译语句（`stmt`），提高效率并防止 SQL 注入。

    3. 绑定参数

       ```C++
       sqlite3_bind_text(stmt, 1, "Alice", -1, SQLITE_STATIC);
       sqlite3_bind_int(stmt, 2, 30);
       
       // SQLITE_STATIC 是 SQLite 绑定参数时的一个回调参数，用于告诉 SQLite：
       // 你传入的字符串是静态的，不需要 SQLite 负责释放内存
       // 比如这里 "Alice" 是常量字符串，SQLite 不会在用完后去释放它。
       ```

       > **作用**：为 SQL 语句中的 `?` 占位符绑定实际参数。

    4. 执行语句

       ```c++
       int rc = sqlite3_step(stmt);
       ```

       > **作用**：执行预编译语句。对于 `INSERT`/`UPDATE`/`DELETE`，执行一次即可；对于 `SELECT`，每次调用返回一行数据

    5. 读取结果（仅 SELECT）

       ```c++
       const unsigned char *name = sqlite3_column_text(stmt, 0);
       int age = sqlite3_column_int(stmt, 1);
       ```

       > **作用**：获取当前行的列数据。

    6. 重置语句

       ```
       sqlite3_reset(stmt);
       ```

       > **作用**：重置语句到初始状态，可以重新绑定参数并再次执行。

    7. 清除绑定

       ```c++
       sqlite3_clear_bindings(stmt);
       ```

       > **作用**：清除所有已绑定的参数。

    8. 释放语句

       ```c++
       sqlite3_finalize(stmt);
       ```

       > **作用**：释放预编译语句对象，防止内存泄漏。

    9. 关闭数据库

       ```c++
       sqlite3_close(db);
       ```

       

