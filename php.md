[返回首页](README.md)
- unset对引用赋值变量的作用
```php
$a = "hello";//定义 $a 变量
$b = &$a;//定义$b是对$a的引用
unset($b);//删除$b变量, 对于$a变量无影响
$b = "world"; //重新定义$b变量
echo $a; //输出$a变量
//输出结果: hello
```
