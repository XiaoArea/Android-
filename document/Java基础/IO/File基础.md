#### File类
Java文件类以抽象的方式代表文件名和目录路径名。该类主要用于文件和目录的创建、文件的查找和文件的删除等。
```
public class File implements Serializable, Comparable<File> {}
```
> 注：File 类只能操作文件的属性，文件的内容是不能操作的。需要配合输入输出流使用；


#### 静态成员

静态成员 | 描述
---|---
String pathSeparator | 与系统相关的路径分隔符字符，为方便起见表示为字符串
char pathSeparatorChar | 与系统相关的路径分隔符字符
String separator | 系统相关的默认名称分隔符字符，为方便起见表示为字符串
char separatorChar | 系统相关的默认名称分隔符字符

> 注：separator 属性，常用于路径分隔，如文件目录的分隔，也是使用最多的；


#### 构造方法

构造方法 | 描述
---|---
File(String pathname) | 通过将给定路径名字符串转换成抽象路径名来创建一个新 File 实例
File(String parent, String child) | 根据 parent 路径名字符串和 child 路径名字符串创建一个新 File 实例
File(File parent, String child) | 通过给定的父抽象路径名和子路径名字符串创建一个新的File实例
File(URI uri) | 通过将给定的 file: URI 转换成一个抽象路径名来创建一个新的 File 实例


#### 常用方法

##### 创建方法
方法 | 描述
---|---
boolean createNewFile() | 创建文件，不存在返回true 已存在返回false
boolean mkdir() | 创建目录，如果上一级目录不存在，则会创建失败
boolean mkdirs() | 创建多级目录，如果上一级目录不存在也会自动创建

##### 删除方法
方法 | 描述
---|---
boolean delete() | 删除文件或目录，如果表示目录，则目录下必须为空才能删除
boolean deleteOnExit() | 文件使用完成后删除

##### 判断方法
方法 | 描述
---|---
boolean canExecute() | 判断文件是否可执行
boolean canRead() | 判断文件是否可读
boolean canWrite() | 判断文件是否可写
boolean exists() | 判断文件或目录是否存在
boolean isDirectory() | 判断此路径是否为一个目录
boolean isFile() | 判断是否为一个文件
boolean isHidden() | 判断是否为隐藏文件
boolean isAbsolute() | 判断是否是绝对路径 文件不存在也能判断

##### 获取方法
方法 | 描述
---|---
String getName() | 获取此路径表示的文件或目录名称
String getPath() | 将此路径名转换为路径名字符串
String getAbsolutePath() | 返回此抽象路径名的绝对形式
String getParent() | 如果没有父目录返回null
long lastModified() | 获取最后一次修改的时间
long length() | 返回由此抽象路径名表示的文件的长度
boolean renameTo(File dest) | 重命名由此抽象路径名表示的文件
File[] liseRoots() | 获取机器盘符
String[] list() | 返回一个字符串数组，命名由此抽象路径名表示的目录中的文件和目录
String[] list(FilenameFilter filter) | 返回一个字符串数组，命名由此抽象路径名表示的目录中满足指定过滤器的文件和目录

#### 具体使用
```
public static void onCreateFile() {
    try {
        // 文件具体目录
        String directoryPath = FILE_PATH + File.separator + "file";
        // 创建目录
        File directoryFile = new File(directoryPath);
        // 判断目录是否存在
        if (!directoryFile.exists()) {
            // 不存在，创建目录
            directoryFile.mkdirs();
        }
        // 创建test.txt文件
        File file = new File(directoryFile, "test.txt");
        // 判断文件是否存在
        if (!file.exists()) {
            // 不存在，创建文件
            file.createNewFile();
        }
        // 写入内容
        Writer writer = new FileWriter(file);
        writer.write("我是测试文件");
        // 刷新并关闭
        writer.flush();
        writer.close();
        // 打印文件信息
        Log.d("StreamUtils", "path: " + file.getPath());
        Log.d("StreamUtils", "absolutePath: " + file.getAbsolutePath());
        Log.d("StreamUtils", "name: " + file.getName());
        Log.d("StreamUtils", "length: " + file.length());
        // 读取内容
        Reader reader = new FileReader(file);
        // 逐个读取字符并打印
        int len;
        while ((len = reader.read()) != -1) {
            Log.d("StreamUtils", "char: " + (char) len);
        }
        // 关闭
        reader.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
###### 输出结果
```
2020-01-10 16:36:31.736 24144-24144/com.example.annotation D/StreamUtils: path: /storage/emulated/0/io/file/test.txt
2020-01-10 16:36:31.736 24144-24144/com.example.annotation D/StreamUtils: absolutePath: /storage/emulated/0/io/file/test.txt
2020-01-10 16:36:31.737 24144-24144/com.example.annotation D/StreamUtils: name: test.txt
2020-01-10 16:36:31.738 24144-24144/com.example.annotation D/StreamUtils: length: 18
2020-01-10 16:36:31.741 24144-24144/com.example.annotation D/StreamUtils: char: 我
2020-01-10 16:36:31.741 24144-24144/com.example.annotation D/StreamUtils: char: 是
2020-01-10 16:36:31.741 24144-24144/com.example.annotation D/StreamUtils: char: 测
2020-01-10 16:36:31.741 24144-24144/com.example.annotation D/StreamUtils: char: 试
2020-01-10 16:36:31.741 24144-24144/com.example.annotation D/StreamUtils: char: 文
2020-01-10 16:36:31.741 24144-24144/com.example.annotation D/StreamUtils: char: 件
```


参考：
- https://www.runoob.com/java/java-file.html
- https://www.cnblogs.com/ysocean/p/6851878.html
