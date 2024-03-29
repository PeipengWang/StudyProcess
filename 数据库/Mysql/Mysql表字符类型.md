# 数据类型
https://www.sjkjc.com/mysql/varchar/
MySQL 中的数据类型包括以下几个大类：

字符串类型
数字类型
日期和时间类型
二进制类型
地理位置数据类型
JSON 数据类型
## MySQL 字符串数据类型

VARCHAR：纯文本字符串，字符串长度是可变的。
CHAR： 纯文本字符串，字符串长度是固定的。当实际字段内容小于定义的长度时，MySQL 会用空白空白符好补足。
VARBINARY：二进制字符串，字符串长度是可变的。
BINARY：二进制字符串，字符串长度是固定的。
TINYTEXT：二进制字符串，最大为 255 个字节。
TEXT：二进制字符串，最大为 65K。
MEDIUMTEXT：二进制字符串，最大为 16M。
LONGTEXT：二进制字符串，最大为 4G。
ENUM：枚举；每个列值可以分配一个 ENUM 成员
SET：集合；每个列值可以分配零个或多个 SET 成员
## MySQL 数字数据类型
数字又是一个常用的数据类型。如果我们要存储年龄、金额等，需要用到数字数据类型。MySQL 支持 SQL 标准中所有的数字类型，包括整数和小数。
下表显示了 MySQL 中数字相关的数据类型：
TINYINT：一个非常小的整数，最大为 1 个字节。
SMALLINT：一个小整数，最大为 2 个字节。
MEDIUMINT：一个中等大小的整数，最大为 3 个字节。
INT：标准整数，最大为 4 个字节。
BIGINT：一个大整数，最大为 8 个字节。
DECIMAL：一个定点数。
FLOAT：单精度浮点数，最大为 4 个字节。
DOUBLE：双精度浮点数，最大为 8 个字节。
BI：按位存储。
## MySQL 日期和时间数据类型
MySQL 提供了丰富的日期和时间类型，这包括日期(DATE)、时间(TIME)、日期和时间(DATETIME)、时间戳(TIMESTAMP)、年份(YEAR)。其中时间戳(TIMESTAMP)数据类型，可以用于跟踪表中一行的变化。
DATE：CCYY-MM-DD 格式的日期值
TIME：hh:mm:ss 格式的时间值
DATETIME：CCYY-MM-DD hh:mm:ss 格式的日期和时间值
TIMESTAMP：CCYY-MM-DD hh:mm:ss 格式的时间戳值
YEAR：CCYY 或 YY 格式的年份值
## MySQL 二进制数据类型
MySQL 还支持存储二进制的数据，比如图片文件等。如果要存储文件，就要用到 BLOB 类型。 BLOB 是 binary large object 的缩写，意思是二进制大对象。
TINYBLOB：最大为 255 个字节。
BLOB：最大为 65K。
MEDIUMBLOB：最大为 16M。
LONGBLOB：最大为 4G。
## MySQL 空间数据类型
GEOMETRY：任何类型的空间值
POINT：使用横坐标和纵坐标表示的一个点
LINESTRING：一条曲线（一个或多个 POINT 值）
POLYGON：一个多边形
GEOMETRYCOLLECTION：GEOMETRY 值的集合
MULTILINESTRING：LINESTRING 值的集合
MULTIPOINT：POINT 值的集合
MULTIPOLYGON：POLYGON 值的集合
## JSON 数据类型
MySQL 从 5.7.8 版本开始支持 JSON 数据类型，允许您更有效地存储和管理 JSON 文档。与 JSON 格式的字符串相比，原生 JSON 数据类型提供有如下的优点：

自动验证。MySQL 会对存储在 JSON 列中的 JSON 文档进行自动验证，无效的文档会产生错误。
最佳存储格式。MySQL 会将存储在 JSON 列中的 JSON 文档转换为允许快速读取文档元素的内部格式。
## MySQL 布尔数据类型
MySQL 没有内置布尔数据类型。但是，MySQL 支持 BOOLEAN 或 BOOL 关键字，MySQL 会将 BOOLEAN 或 BOOL 类型转换为 TINYINT(1)。当我们插入 TRUE 或者 FALSE 时，MySQL 会存储为 1 或者 0。


## 常用的类型解析
### VARCHAR 语法
当我们使用 VARCHAR 数据类型时，我们需要指定一个最大的长度。其语法如下：
VARCHAR(max_length)
其中 max_length 是一个数值，它指示了此列的最大字符数。如果我们不指定此值，则默认值是 255。也就是说 VARCHAR 等同于 VARCHAR(255)。 VARCHAR 最多长度为 65535 个字节。
MySQL 存储 VARCHAR 数值时，会将最前的 1 或者 2 个字节存储为实际字符串内容的长度。如果列的值少于 255 个字节，则长度前缀为 1 个字节，否则为 2 个字节。

### int用法
MySQL INT 类型使用起来简单，如下：
INT [UNSIGNED]
这里： UNSIGNED 属性标识了此数据类型为无符号整数。
有符号取值范围为-2147483648~2147483647
无符号：0~4294967295

### DECIMAL 类型介绍
 DECIMAL 数据类型是定点数数据类型，用来存储精确的树枝，比如账务金额等。底层实现上，MySQL 使用二进制形式存储该类型的值
 为了存储精确的数值，我们需要为 DECIMAL 数据类型指定总位数和小数位数。这里是 DECIMAL 数据类型的语法：
DECIMAL[(M[,D])] [UNSIGNED] [ZEROFILL]
这里：
M 是总的位数，不包含小数点和正负号。
D 是小数部分的位数。如果 D 为 0 则表示没有小数部分。当 D 省略时，默认值为 0。
UNSIGNED 属性表示数值是无符号的。无符号的数字不能是负数。
ZEROFILL 属性表示当整数部分位数不足时，用整数的左侧用 0 填充。带有 ZEROFILL 的列将自动具有 UNSIGNED 属性。这和 INT 数据类型一样。
比如，我们定义了如下一个列：
amount DECIMAL(9, 4);
那么 amount 列的值的范围是从 -99999.9999 到 99999.9999。
### text类型





