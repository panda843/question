# 例子
![实例图](https://upload-images.jianshu.io/upload_images/3369258-e1798cc0cb109185.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

# 位置说明
![权限位置说明](https://upload-images.jianshu.io/upload_images/3369258-c196788e7b51224d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/468/format/webp)

# 第一个字段
- \- 代表文件
- d 代表目录
- l 代表链接
- c 代表字符型设备
- b 代表块设备
- n 代表网络设备

# 权限值说明
|权限|	权限数值|	二进制|	具体作用|
|   ------ |   ------  |  ------  |  ------  |
|r	|4|	00000100|	read，读取。当前用户可以读取文件内容，当前用户可以浏览目录。|
|w	|2|	00000010|	write，写入。当前用户可以新增或修改文件内容，当前用户可以删除、移动目录或目录内文件。|
|x	|1|	00000001|	execute，执行。当前用户可以执行文件，当前用户可以进入目录。|