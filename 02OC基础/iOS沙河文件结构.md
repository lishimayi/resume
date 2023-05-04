# iOS沙河文件结构

沙盒机制 只能访问程序自己的目录，每个app特有的文件夹。

### document 

可以进行本分和恢复，体积较大，一般存档用户数据。

### Library

开发者常用的文件夹，可以自定义子文件夹

### tmp

临时文件，不会备份，启动时有可能被清除。

### systemdata

## NSPathUtilities
