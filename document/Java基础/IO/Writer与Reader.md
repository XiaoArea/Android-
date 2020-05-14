## 字符流Writer与Reader

#### 思考

##### 1. 为什么要使用字符流？

> 因为使用字节流操作汉字或特殊符号语言的时候容易乱码，因为汉字不止一个字节，为了解决这个问题，建议使用字符流。
##### 2. 什么情况下使用字符流？

> 一般可以用记事本打开的文件，可以看到内容不乱码的。就是文本文件，可以使用字符流。而操作二进制文件（比如图片、音频、视频）必须使用字节流。

---

#### 字符输出流：Writer

```
public abstract class Writer implements Appendable, Closeable, Flushable {}
```

#### 子类
![子类结构图](https://note.youdao.com/yws/public/resource/4f2991b25ab7504fe4a24a4bdfaff4dc/xmlnote/WEBRESOURCE1cbe560951fd03bd806540b88ff73af4/5315)

#### 基础方法

方法 | 描述
---|---
void write(int c) | 写入单个字符
void write(char[] cbuf) | 写入字符数组
void write(char[] cbuf, int off, int len) | 写入字符数组指定部分
void write(String str) | 写入字符串
void write(String str, int off, int len) | 写入字符串中指定部分
Writer append(char c) | 将指定字符添加到此Writer
Writer append(CharSequence csq) | 将指定字符序列添加到此Writer
Writer append(CharSequence csq, int start, int end) | 将指定字符序列的子序列添加到此Writer
void flush() | 刷新该流的换冲
void close() | 关闭该流，需要先刷新它

> 注：再进行写的操作的时候，即w.write()方法时，用完该方法最好刷新一下流，即w.flush()，否则流中的数据会有损失；关流操作的close()方法会在执行关流操作之前执行一次flush()方法

#### 基础使用

```
public static void onWriter() {
    try {
        // 文件存储路径
        String path = FILE_PATH + File.separator + "character";
        // 创建文件夹
        File folder = new File(path);
        if (!folder.exists()) {
            folder.mkdirs();
        }
        // 创建文件
        File file = new File(path, "test.txt");
        if (!file.exists()) {
            file.createNewFile();
        }
        // 创建字符输出流Writer
        Writer writer = new FileWriter(file);
        // 进行IO写入操作
        writer.write(65); // 写入A
        writer.write("chars".toCharArray()); // 写入字符数组
        writer.write("ABCDEFG".toCharArray(), 1, 5); // 写入字符数组指定大小
        writer.write("字符串"); // 写入字符串
        writer.write("字符串", 1, 2); // 写入字符串指定内容
        // 刷新流
        writer.flush();
        // 关闭流
        writer.close();
        // 打印日志
        Log.d("StreamUtils", file.getAbsolutePath());
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

---

#### 字符输入流：Reader

```
public abstract class Reader implements Readable, Closeable {}
```

#### 子类
![子类结构图](https://note.youdao.com/yws/public/resource/4f2991b25ab7504fe4a24a4bdfaff4dc/xmlnote/WEBRESOURCE6ddc1665eb485fc4aa558581e5798100/5318)

#### 基础方法

方法 | 描述
---|---
int read() | 读取单个字符
int read(char[] cbuf) | 将字符读入数组
int read(char[] cbuf, int off, int len) | 将字符读入数组的某一部分
int read(CharBuffer target) | 试图将字符读入指定的字符缓冲区
long skip(long n) | 跳过字符
boolean ready() | 判断是否准备读取此流
boolean markSupported() | 判断此流是否支持mark()操作
void mark(int readAheadLimit) | 标记流中的当前位置
void reset() | 重置该流
void close() | 关闭该流并释放与之关联的所有资源

> 注：mark()与reset()结合使用

#### 基础使用

```
public static void onReader() {
    try {
        // 文件存储路径
        String path = FILE_PATH + File.separator + "character";
        // 获取文件
        File file = new File(path, "test.txt");
        // 判断文件是否存在
        if (file.exists()) {
            // 创建字符输入流Reader
            Reader reader = new FileReader(file);
            // 包装流用于标记，重置
            BufferedReader bufferedReader = new BufferedReader(reader);
            // 标记当前位置
            // 参数 readAheadLimit - 在仍保留该标记的情况下，对可读取字符数量的限制。在读取达到或超过此限制的字符后，
            // 尝试重置流可能会失败。限制值大于输入缓冲区的大小将导致分配一个新缓冲区，其大小不小于该限制值。
            // 因此应该小心使用较大的值。
            bufferedReader.mark((int) file.length());
            // 进行IO读取操作 reader.read()
            // 每次读取一个字符，读到最后返回 -1
            int len;
            while ((len = bufferedReader.read()) != -1) {
                Log.d("StreamUtils", "单个字符:" + (char) len);
            }
            // 重置
            bufferedReader.reset();
            // 将字符读进字符数组
            char[] cbuf = new char[10];
            while ((len = bufferedReader.read(cbuf)) != -1) {
                Log.d("StreamUtils", "1字符数组:" + new String(cbuf, 0, len));
            }
            // 重置
            bufferedReader.reset();
            // 将字符读进字符数组指定范围
            while ((len = bufferedReader.read(cbuf, 0, 5)) != -1) {
                Log.d("StreamUtils", "2字符数组:" + new String(cbuf, 0, len));
            }
            // 关闭流
            bufferedReader.close();
            reader.close();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

参考：https://www.cnblogs.com/ysocean/p/6859242.html
