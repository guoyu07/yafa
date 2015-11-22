PHP_WUF
====

扩展补充了部分 [Yaf]() 未提供的常用功能。

MySQLi 包装
---
``` php
class wuf_database_mysql {
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