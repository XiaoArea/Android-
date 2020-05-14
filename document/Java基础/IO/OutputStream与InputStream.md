#### 字节输出流：OutputStream
OutputStream是一个抽象类，是所有字节输出流类的超类。
```
public abstract class OutputStream implements Closeable, Flushable {}
```

##### 子类

![OutputStream子类结构图](https://note.youdao.com/yws/public/resource/4f2991b25ab7504fe4a24a4bdfaff4dc/xmlnote/WEBRESOURCEcfa47ce6868626836ae49ca5e6c328f8/5096)

##### 基础方法

方法 | 描述
---|---
void write(int b) | 往流中写一个字节b 
void write(byte b[]) | 往流中写一个字节数组b 
void write(byte b[], int off, int len) | 把字节数组b中从下标off开始，长度为len的字节写入流中
void flush() | 刷新输出流并强制缓存的字节写入输出流中
void close() | 关闭输出流并释放与流相关联的任何系统资源

> 注：flush()方法由于某些流支持缓存功能，该方法将把缓存中所有内容强制输出到流中。

##### 基本使用

```
public static void main(String[] strs) {
    try {
        // 1.创建目标对象文件，不写盘符，默认该文件是在该项目的根目录下。
        File target = new File("test.txt");
        // 判断该文件是否存在
        if (!target.exists()) {
            // 创建文件
            target.createNewFile();
        }
        // 2.创建文件的字节输出流对象,第二个参数是 Boolean 类型，true 表示后面写入的文件追加到数据后面，false 表示覆盖
        OutputStream out = new FileOutputStream(target, true);
        // 3.具体的 IO 操作(将数据写入到文件 test.txt 中)

        // write(int b): 往流中写一个字节b
        // write(byte[] b): 往流中写一个字节数组b
        // write(byte[] b, int off, int len): 把字节数组b中从下标off开始，长度为len的字节写入流中

        out.write(65); //将 A 写入到文件中
        out.write("Aa".getBytes()); //将 Aa 写入到文件中
        out.write("ABCDEFG".getBytes(), 1, 5); //将 BCDEF 写入到文件中
        // 经过上面的操作，test.txt 文件中数据为 AAaBCDEF

        // 4.关闭流资源
        out.close();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```


---

#### 字节输入流：InputStream
InputStream是一个抽象类,是所有字节输入流类的超类。
```
public abstract class InputStream implements Closeable {}
```

##### 子类

![InputStream子类结构图](https://note.youdao.com/yws/public/resource/4f2991b25ab7504fe4a24a4bdfaff4dc/xmlnote/WEBRESOURCE87f9453f06da41603fd0f629f7cdafa3/5094)

##### 基础方法

方法 | 描述
---|---
int read() | 从输入流读取数据的下一个字节
int read(byte b[]) | 从输入流读取一段字节数，并将它们存储至b数组中
int read(byte b[], int off, int len) | 从输入流读取最多len字节的数据到一个字节数组
int available() | 返回输入流中尚未读取的字节的数量(估值)
long skip(long n) | 读指针跳过n个字节不读，返回值为实际跳过的字节数量 
void mark(int readlimit) | 记录当前读指针所在位置，readlimit表示读指针读出readlimit个字节后所标记的指针位置才失效
void reset() | 把读指针重新指向用mark方法所记录的位置
boolean markSupported() | 判断该输入流是否支持mark和reset方法
void close() | 关闭输入流并释放与流相关联的任何系统资源

##### 基本使用

```
public static void main(String[] strs) {
    try {
        // 1.创建目标对象，输入流表示那个文件的数据保存到程序中。不写盘符，默认该文件是在该项目的根目录下
        // test.txt 保存的文件内容为：AAaBCDEF
        File target = new File("test.txt");
        // 2.创建输入流对象
        InputStream in = new FileInputStream(target);
        // 3.具体的 IO 操作（读取 test.txt 文件中的数据到程序中）

        // 注意：读取文件中的数据，读到最后没有数据时，返回-1
        // int read():读取一个字节，返回读取的字节
        // int read(byte[] b):读取多个字节,并保存到数组 b 中，从数组 b 的索引为 0 的位置开始存储，返回读取了几个字节
        // int read(byte[] b,int off,int len):读取多个字节，并存储到数组 b 中，从数组b 的索引为 0 的位置开始，长度为len个字节

        // int read():读取一个字节，返回读取的字节
        int data1 = in.read();// 获取 test.txt 文件中的数据的第一个字节
        System.out.println((char) data1); //A
        // int read(byte[] b):读取多个字节保存到数组b 中
        byte[] buffer = new byte[10];
        in.read(buffer);// 获取 test.txt 文件中的前10 个字节，并存储到 buffer 数组中
        System.out.println(Arrays.toString(buffer)); // [65, 97, 66, 67, 68, 69, 70, 0, 0, 0]
        System.out.println(new String(buffer)); // AaBCDEF[][][]

        // int read(byte[] b,int off,int len):读取多个字节，并存储到数组 b 中,从索引 off 开始到 len
        in.read(buffer, 0, 3);
        System.out.println(Arrays.toString(buffer)); // [65, 97, 66, 0, 0, 0, 0, 0, 0, 0]
        System.out.println(new String(buffer)); // AaB[][][][][][][]
        // 4.关闭流资源
        in.close();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

参考：https://www.cnblogs.com/ysocean/p/6854541.html
