当操作符与不同类型的操作数一起使用时，会发生类型转换以使操作数兼容。则会发生转换隐式

# 隐式类型转换规则
- 如果一个或两个参数都是NULL，比较的结果是NULL，除了NULL安全的<=>相等比较运算符。对于NULL <=> NULL，结果为true。不需要转换
- 如果比较操作中的两个参数都是字符串，则将它们作为字符串进行比较。
- 如果两个参数都是整数，则将它们作为整数进行比较。
- 如果不与数字进行比较，则将十六进制值视为二进制字符串
- 如果其中一个参数是十进制值，则比较取决于另一个参数。 如果另一个参数是十进制或整数值，则将参数与十进制值进行比较，如果另一个参数是浮点值，则将参数与浮点值进行比较
- 如果其中一个参数是TIMESTAMP或DATETIME列，另一个参数是常量，则在执行比较之前将常量转换为时间戳。
- 在所有其他情况下，参数都是作为浮点数（实数）比较的。

# 使用CAST函数显示转换
```sql
SELECT 38.8, CAST(38.8 AS CHAR);
```


# 字段为varchar类型,查询值为int类型会走索引吗?
- 不会(因为MySQL在对文本类型和数字类型进行比较的时候会进行隐式的类型转换,其他类型同理)

# int类型 使用unix_timestamp查询 走索引吗?
- 会

# 原因说明
以下是5.5官方手册的说明
```
If both arguments in a comparison operation are strings, they are compared as strings.
两个参数都是字符串，会按照字符串来比较，不做类型转换。
If both arguments are integers, they are compared as integers.
两个参数都是整数，按照整数来比较，不做类型转换。
Hexadecimal values are treated as binary strings if not compared to a number.
十六进制的值和非数字做比较时，会被当做二进制串。
If one of the arguments is a TIMESTAMP or DATETIME column and the other argument is a constant, the constant is converted to a timestamp before the comparison is performed. This is done to be more ODBC-friendly. Note that this is not done for the arguments to IN()! To be safe, always use complete datetime, date, or time strings when doing comparisons. For example, to achieve best results when using BETWEEN with date or time values, use CAST() to explicitly convert the values to the desired data type.
有一个参数是 TIMESTAMP 或 DATETIME，并且另外一个参数是常量，常量会被转换为 timestamp
If one of the arguments is a decimal value, comparison depends on the other argument. The arguments are compared as decimal values if the other argument is a decimal or integer value, or as floating-point values if the other argument is a floating-point value.
有一个参数是 decimal 类型，如果另外一个参数是 decimal 或者整数，会将整数转换为 decimal 后进行比较，如果另外一个参数是浮点数，则会把 decimal 转换为浮点数进行比较
In all other cases, the arguments are compared as floating-point (real) numbers.
所有其他情况下，两个参数都会被转换为浮点数再进行比较
```