# 处理文件错误的机制上面不同
- require() :如果文件不存在，会报出一个fatal error.脚本停止执行;
- include() : 如果文件不存在，会给出一个 warning，但脚本会继续执行;

# php性能
- 对include()来说，在include()执行时文件每次都要进行读取和评估;
- 对require()来说，文件只处理一次(实际上，文件内容替换了require()语句)。

# 不同的使用弹性
- require的使用方法如 require("./inc.php"); 。通常放在PHP程式的最前面，PHP程式在执行前，就会先读入require所指定引入的档案，使它变成PHP 程式网页的一部份。
- include使用方法如 include("./inc.php"); 。一般是放在流程控制的处理区段中。PHP程式网页在读到 include的档案时，才将它读进来。这种方式，可以把程式执行时的流程简单化。

# once
- 带once和不带once的区别主要是:带once的会判断你在加载这个文件之前是否已经加载过了文件，避免重复加载。