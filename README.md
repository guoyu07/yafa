YAFA
====

仅支持 PHP7。扩展补充了部分 [Yaf](https://github.com/laruence/yaf/tree/php7) 未提供的常用功能。

MySQLi 包装
---
依赖 mysqli 系统内置扩展。

``` php
class yafa_database_mysql {
	/**
	 * 数据库配置初始化（建议在 Yaf_Bootstrap_Abstract 实现类中调用）
	 * $config 包含数据库配置，形式如下：
	 * [
	 *	"master"=>[
	 *		["host"=>"127.0.0.1", "port"=>3306, "user"=>"root", "pass"=>"123456", "db"=>"test"],
	 *		["host"=>"127.0.0.1", "port"=>3306, "user"=>"root", "pass"=>"123456", "db"=>"test"],
	 *	],
	 *	"slave"=>[
	 * 		["host"=>"127.0.0.1", "port"=>3306, "user"=>"root", "pass"=>"123456", "db"=>"test"]
	 *	]
	 * ]
	 */
	public static function init($config) : bool;
	/**
	 * 获取对应上述主库配置 $index 的数据库连接对象
	 */
	public static function get_master($index = 0) : wuf_database_mysql;
	/**
	 * 获取对应上述从库配置 $index 的数据库连接对象
	 */
	public static function get_slave($index = 0) : wuf_database_mysql;
	/**
	 * 包裹 `mysqli`，一般使用上述 `get_master()` | `get_slave()` 方法即可；
	 * 特殊情况用户构建 `mysqli` 实例并调用
	 */
	public function __construct(mysqli $mysqli);
	/**
	 * 根据 `$format` 生成 SQL，提供类似 `mysqli_stmt::bind_param()` 形式的文本替换
	 * 替换参数会经过 `mysqli::escape_string()` 转义
	 * @example $db->format("INSERT INTO `table` VALUES(?,?)", "abc", "xx'xx");
	 *  => "INSERT INTO `table` VALUES('abc','xx\'xx')"
	 * 注：不要直接用于二进制数据生成 SQL
	 */
	public function format(string $format, ...) : string;
	/**
	 * 调用 `format()` 生成 SQL 语句，然后调用 `mysqli::query()` 并返回
	 * @return msyqli_result | false
	 */
	public function format_query(string $format, ...);
	/**
	 * $data 指定要保存的数据
	 * 注：$data 如果存在 二级 数组（类似 $data = ['a'=>['1','2','3'], 'b'=>'xxx'];）将自动使用 `json_encode` 进行序列化
	 */
	public function insert(string $table, array $data): bool;
	/**
	 * 请参考 `select`
	 */
	public function remove(string $table, array|string $cond): bool;
	/**
	 * 请参考 `select`
	 */
	public function update(string $table, array $data, array|string $cond): bool;
	/**
	 * 查询字段
	 * @example $column = ["c1", "c2"];
	 * @example $column = "*";
	 * 查询条件
	 * @example $cond   = ["a"=>"xxx", "b[!]"=>"xxx", "c[<>]"=>[100, 200], "d[><]"=>[300, 400]];
	 * => "WHERE `a` = 'xxx' AND `b` != 'xxx' AND `c` BETWEEN 100 AND 200 AND `d` NOT BETWEEN 300 AND 400"
	 * @example $cond   = ["e"=>null, "f[!]"=>null, "g"=>["X", "Y", "Z"], "h[!]"=>["X", "Y", "Z"]];
	 * => "WHERE `e` IS NULL AND `f` IS NOT NULL AND `g` IN ('X', 'Y', 'Z') AND `h` NOT IN ('X', 'Y', 'Z')"
	 * @example $cond   = ["i[~]"=>"%xxx", "j[!~]"=>"%xxx"];
	 * => "WHERE `i` LIKE '%xxx' AND `j` NOT LIKE '%xxx'"
	 * @example $cond   = ["a"=>"xxx", "OR"=>["b"=>"xxx", "c"=>"xxx"]];
	 * => "WHERE `a` = 'xxx' AND (`b` = 'xxx' OR `c`= 'xxx')"
	 * @example $cond   = ["OR"=>["b"=>"xxx", "c"=>"xxx"]];
	 * => "WHERE `b` = 'xxx' OR `c`= 'xxx'"
	 * 归类
	 * @example $group = "`a`";
	 * => "GROUP BY `a`"
	 * @example $group = ["a", "b"];
	 * => "GROUP BY `a`,`b`"
	 * 排序
	 * @example $order = "`a` ASC, `b` DESC";
	 * => "ORDER BY `a` ASC, `b` DESC",
	 * @example $order = ["a"=>true, "b"=>"DESC", "c"=>false]
	 * => "ORDER BY `a` ASC, `b` DESC, `c` DESC"
	 * 截取
	 * @example $limit = 10;
	 * => "LIMIT 10"
	 * @example $limit = "0,10";
	 * => "LIMIT 0,10"
	 * @example $limit = [0, 10];
	 * => "LIMIT 0, 10"
	 * 注意：为防止误操作 `select` 不会自动进行 json 的反序列化
	 */
	public function select(string $table, array|string $column, array|string $cond, array|string $group, array|string $order, array|string|integer $limit);
	/**
	 * 获取一条数据，功能基本类似：
	 * `$mysqli->query("SELECT ....... LIMIT 1")->fetch_assoc()`;
	 */
	public function one(string $table, array|string $cond, array|string $order);
	/**
	 * ... 其他访问带地道 mysqli
	 */
}
```

[PhpRedis](https://github.com/phpredis/phpredis) 配置协助
---
**依赖 phpredis 扩展。**
注：从 `get_master` | `get_slave` 返回的即就是 `Redis` 类型，非包装类型。

``` php
class yafa_database_redis {
	/**
	 * $config 包含数据库配置，形式如下：
	 * [
	 *	"master"=>[
	 *		["host"=>"127.0.0.1", "port"=>3306, "auth"="passwd", "prefix"=>"pre_"],
	 *		["host"=>"127.0.0.1", "port"=>3306, "auth"="passwd", "prefix"=>"pre_"],
	 *	],
	 *	"slave"=>[
	 * 		["host"=>"127.0.0.1", "port"=>3306, "auth"="passwd", "prefix"=>"pre_"],
	 *	]
	 * ]
	 * 当配置中存在 "auth" 将会在实例化 phpredis 对象后，调用 `auth($auth)` 方法；
	 * 当配置中存在 "prefix" 将会在实例化 phpredis 对象后，调用 `setOption(Reids::OPT_PREFIX, $prefix)`
	 */
	public static function init($config) : bool;
	public static function get_master($index = 0) : Redis;
	public static function get_slave($index = 0)  : Redis;
	// 其他方法代理到 PhpRedis 实例，并对第一个参数添加 prefix 前缀
}
```

[SSDB](https://github.com/ideawu/ssdb) 扩展客户端
---

``` php
class yafa_database_ssdb {
	/**
	 * $config 包含数据库配置，形式如下：
	 * [
	 *	"master"=>[
	 *		["host"=>"127.0.0.1", "port"=>3306, "auth"="passwd", "prefix"=>"pre_"],
	 *		["host"=>"127.0.0.1", "port"=>3306, "auth"="passwd", "prefix"=>"pre_"],
	 *	],
	 *	"slave"=>[
	 * 		["host"=>"127.0.0.1", "port"=>3306, "auth"="passwd", "prefix"=>"pre_"],
	 *	]
	 * ]
	 * 当配置中存在 "auth" 将会在实例化 phpredis 对象后，调用 `auth($auth)` 方法；
	 * 当配置中存在 "prefix" 将会对所有 KEY 进行附加前缀操作
	 */
	public static function init($config) : bool;
	public static function get_master($index = 0) : yafa_database_ssdb;
	public static function get_slave($index = 0)  : yafa_database_ssdb;
	/**
	 * 实现对 ssdb 调用的包装，目前与 PHP 版本的客户端支持命令一致
	 */
	public function __call($cmd, $args) : mixed;
}
```

注：SSDB 提供的 CPP 客户端 静态库 无法直接包装到扩展中，需要一些编译方面的调整，同时为了 Web 项目考虑增加了 数据接收 timeout 设置功能；
> 
> 1. 使用 build.sh 生成 build_config.mk 后停止编译；
> 2. 在 src/util 下：
> ``` bash
> make bytes.o CFLAGS="-D__STDC_FORMAT_MACROS -fPIC"
> ```
> 3. 在 src/net 下：
> ``` bash
> make link.o CFLAGS="-D__STDC_FORMAT_MACROS -fPIC"
> ```
> 4. src/client/SSDB_client.h 修改 connect 加入 timeout 参数:
> ``` cpp
> static Client* connect(const char *ip, int port, int timeout = 2000);
> static Client* connect(const std::string &ip, int port, int timeout = 2000);
> ```
> 5. src/client/SSDB_impl.cpp 对应给 connect 函数加上 timeout 参数并在完成连接后增加：
> ``` cpp
> // [-------
> struct timeval to;  
> to.tv_sec = timeout / 1000;
> to.tv_usec = timeout * 1000 % 1000000;
> setsockopt(client->link->fd(), SOL_SOCKET, SO_RCVTIMEO, &to, sizeof(to));
> // ------]
> return client;
> ```
> 6. 在 src/client 下 make CFLAGS="-fPIC" 得到静态库

