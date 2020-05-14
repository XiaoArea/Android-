## 简介
APT(Annotation Processing Tool)即注解处理器，是一种处理注解的工具，确切的说它是javac的一个工具，它用来在编译时扫描和处理注解。注解处理器以Java代码(或者编译过的字节码)作为输入，生成.java文件作为输出。

简单来说就是在编译期，通过注解生成.java文件。

## 作用
使用APT的优点就是方便、简单，可以少些很多重复的代码。根据规则，帮我们生成代码、生成类文件,如ButterKnife、Dagger、EventBus等注解框架。

##### Element 程序元素
元素 | 描述
---|---
PackageElement | 表示一个包程序元素，提供对有关包及其成员的信息的访问
ExecutableElement | 表示某个类或接口的方法、构造方法或初始化程序（静态或实例）
TypeElement | 表示一个类或接口程序元素，提供对有关类型及其成员的信息的方法
VaribaleElement | 表示一个字段、enum常量、方法或构造方法参数、局部变量或异常参数

##### 常用API

方法 | 描述
---|---
getEnclosedElements() | 返回该元素直接包含的子元素
getEnclosingElement() | 返回包含该element的父element，与上一个方法相反
getKind() | 返回element的类型，判断是那种element
getModifiers() | 获取修饰关键字，入public static final 等关键字
getSimpleName() | 获取名字，不带包名
getQualifiedName() | 获取全名，如果是类的话，包含完整的包名路径
getParameters() | 获取方法的参数元素，每个元素是一个VariableElement
getReturnType() | 获取方法元素的返回值
getConstantValue() | 如果属性变量被final修饰，则可以使用该方法获取它的值


## Demo
#### 项目结构
- 创建Android Module命名为app 
- 创建Java library Module命名为 apt-annotation 存放自定义注解
- 创建Java library Module命名为 apt-compiler 依赖 apt-annotation、auto-service 指定注解处理器
- 创建Android library Module 命名为 apt-library 存放工具类

> ==注==：为什么两个模块一定要是Java Library？如果创建Android Library模块会发现不能找到AbstractProcessor这个类，这是因为Android平台是基于OpenJDK的，而OpenJDK中不包含APT的相关代码。因此，在使用APT时，必须在Java Library中进行。

#### 环境配置
##### apt-annotation 对应的 build.gradle 进行配置
```
dependencies {
    ...
    // DES: 导入annotation库, 因lib中使用到了内置注解
    implementation 'androidx.annotation:annotation:1.0.0@jar'
}
// DES: 指定编码格式
tasks.withType(JavaCompile) {
    options.encoding = "UTF-8"
}
// DES: 指定Java JDK 兼容及目标版本
// DES: 7 表示 Java JDK 1.7
sourceCompatibility = "7"
targetCompatibility = "7"
```
##### apt-compiler 对应的 build.gradle 进行配置
```
dependencies {
    ...
    // DES: 依赖自定义注解
    implementation project(":apt-annotation")
    // DES: 导入官方APT处理Service
    compileOnly 'com.google.auto.service:auto-service:1.0-rc4'
    annotationProcessor 'com.google.auto.service:auto-service:1.0-rc4'

    // DES: 导入JavaPoet库用于类调用的形式来生成Java代码
    implementation 'com.squareup:javapoet:1.9.0'
}
// DES: 指定编码格式
tasks.withType(JavaCompile) {
    options.encoding = "UTF-8"
}
// DES: 指定Java JDK 兼容及目标版本
// DES: 7 表示 Java JDK 1.7
sourceCompatibility = "7"
targetCompatibility = "7"
```
###### 版本问题
```
// Android Studio 3.3.2 + Gradle 4.10.1 （临界版本）
// 注册注解，并对其生成META-INF的配置信息，rc2在gradle 5.1.1后有坑
// AS3.3.2 + Gradle 4.10.1 + auto-service:1.0-rc2
implementation 'com.google.auto.service:auto-service:1.0-rc2'

// Android Studio 3.4.1 + Gradle 5.1.1（向下兼容）
// AS3.4.1 + Gradle 5.1.1 + auto-service:1.0-rc4
annotationProcessor 'com.google.auto.service:auto-service:1.0-rc4'
compileOnly 'com.google.auto.service:auto-service:1.0-rc4'
```

##### apt-library 无需依赖

##### app 对应的 build.gradle 进行配置
```
dependencies {
    ...
    // 导入library
    implementation project(":apt-library")
    // 导入自定义注解
    implementation project(":apt-annotation")
    // 指定注释处理器
    annotationProcessor project(":apt-compiler")
    ...
}
```
#### 相关代码
##### apt-annotation 自定义注解

```
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
/**
 * 自定义注解 @BindView 牛油刀
 * @author XGY
 */
@Target(ElementType.FIELD) // 作用于成员属性上
@Retention(RetentionPolicy.RUNTIME) // 指定为运行时
public @interface BindView {
    int value();
}
```

---

##### apt-compiler 
###### 节点信息
```
/**
 * 节点信息Bean
 * @author XGY
 */
public class NodeInfo {
    /** 包路径 */
    private String packageName;
    /** 节点所在类名称 */
    private String className;
    /** 节点类型名称 */
    private String typeName;
    /** 节点名称 */
    private String nodeName;
    /** 注解的value */
    private int value;

    public NodeInfo(String packageName, String className, String typeName, 
                        String nodeName, int value) {
        this.packageName = packageName;
        this.className = className;
        this.typeName = typeName;
        this.nodeName = nodeName;
        this.value = value;
    }
    public String getPackageName() { return packageName; }
    public String getClassName() { return className; }
    public String getTypeName() { return typeName; }
    public String getNodeName() { return nodeName; }
    public int getValue() { return value; }
}
```
###### 处理器

```
import com.example.annotation.BindView;
import com.example.compiler.bean.NodeInfo;
import com.google.auto.service.AutoService;

import java.io.IOException;
import java.io.Writer;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Set;

import javax.annotation.processing.AbstractProcessor;
import javax.annotation.processing.Filer;
import javax.annotation.processing.Messager;
import javax.annotation.processing.Processor;
import javax.annotation.processing.ProcessingEnvironment;
import javax.annotation.processing.RoundEnvironment;
import javax.annotation.processing.SupportedAnnotationTypes;
import javax.annotation.processing.SupportedSourceVersion;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.Element;
import javax.lang.model.element.TypeElement;
import javax.lang.model.util.Elements;
import javax.lang.model.util.Types;
import javax.tools.Diagnostic;
import javax.tools.JavaFileObject;

/**
 * 自定义注解 @BindView 编译器
 * @author XGY
 */
@AutoService(Processor.class) // 指定注解处理器
@SupportedAnnotationTypes("com.example.annotation.BindView") // 指定需要处理的注解
@SupportedSourceVersion(SourceVersion.RELEASE_7) // 指定Java JDK编译版本
public class BindViewProcessor extends AbstractProcessor {

    /** Element操作类 */
    private Elements mElementUtils;
    /** 类信息工具类 */
    private Types mTypeUtils;
    /** 日志工具类 */
    private Messager mMessager;
    /** 文件创建工具类 */
    private Filer mFiler;

    /** 节点信息缓存 */
    private Map<String, List<NodeInfo>> mCache = new HashMap<>();

    /** 初始化 */
    @Override
    public synchronized void init(ProcessingEnvironment environment) {
        super.init(environment);
        // 获取相关工具
        mElementUtils = environment.getElementUtils();
        mTypeUtils = environment.getTypeUtils();
        mMessager = environment.getMessager();
        mFiler = environment.getFiler();

        // 打印Build提示
        mMessager.printMessage(Diagnostic.Kind.NOTE, "开始处理自定义 @BindView 注解");
    }

    /** 处理注解 */
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        // 判断是否有使用 @BindView 注解
        if (annotations != null && !annotations.isEmpty()) {
            // 获取所有 @BindView 节点
            Set<? extends Element> elements = roundEnv.getElementsAnnotatedWith(BindView.class);
            // 判断节点集合不为空
            if (elements != null && !elements.isEmpty()) {
                // 循环遍历节点
                for (Element element : elements) {
                    // 获取节点包信息
                    String packageName = mElementUtils.getPackageOf(element).getQualifiedName().toString();
                    // 获取节点类信息，由于 @BindView 作用于成员属性上，所以这里使用 getEnclosingElement() 获取父节点信息
                    String className = element.getEnclosingElement().getSimpleName().toString();
                    // 获取节点类型
                    String typeName = element.asType().toString();
                    // 获取节点标记的属性名称
                    String simpleName = element.getSimpleName().toString();
                    // 获取注解的值
                    int value = element.getAnnotation(BindView.class).value();

                    // 打印
                    mMessager.printMessage(Diagnostic.Kind.NOTE, "packageName：" + packageName);
                    mMessager.printMessage(Diagnostic.Kind.NOTE, "className：" + className);
                    mMessager.printMessage(Diagnostic.Kind.NOTE, "typeName：" + typeName);
                    mMessager.printMessage(Diagnostic.Kind.NOTE, "simpleName：" + simpleName);
                    mMessager.printMessage(Diagnostic.Kind.NOTE, "value：" + value);

                    // 缓存KEY
                    String key = packageName + "." + className;
                    // 缓存节点信息
                    List<NodeInfo> nodeInfos = mCache.get(key);
                    // 判断是否为空
                    if (nodeInfos == null) {
                        // 初始化
                        nodeInfos = new ArrayList<>();
                        // 载入
                        nodeInfos.add(new NodeInfo(packageName, className, typeName, simpleName, value));
                        // 缓存
                        mCache.put(key, nodeInfos);
                    } else {
                        // 载入
                        nodeInfos.add(new NodeInfo(packageName, className, typeName, simpleName, value));
                    }
                }
                // 判断临时缓存是否不为空
                if (!mCache.isEmpty()) {
                    // 遍历临时缓存文件
                    for (Map.Entry<String, List<NodeInfo>> stringListEntry : mCache.entrySet()) {
                        try {
                            // 创建文件
                            createFile(stringListEntry.getValue());
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }

                return true;
            }
        }

        return false;
    }

    /** 创建Java文件 */
    private void createFile(List<NodeInfo> infos) throws IOException {
        // 获取第一个节点用来获取信息
        NodeInfo info = infos.get(0);
        // 生成的类名称
        String className = info.getClassName() + "$$ViewBinding";
        // JavaFileObject
        JavaFileObject file = mFiler.createSourceFile(info.getClassName() + "." + className);
        // Writer
        Writer writer = file.openWriter();
        // 设置包路径
        writer.write("package " + info.getPackageName() + ";\n\n");
        // 设置类名称
        writer.write("public class " + className + " {\n\n");
        writer.write("\tpublic static void bind(" + info.getClassName() + " target) {\n");
        // 循环遍历设置方法体
        for (NodeInfo node : infos) {
            writer.write("\t\ttarget." + node.getNodeName() + " = (" + node.getTypeName() +
                    ") target.findViewById(" + node.getValue() + ");\n");
        }
        writer.write("\t}\n");
        writer.write("}");
        // 关闭
        writer.close();
    }
}
```
==注意：== 

```
// 指定注解处理器
@AutoService(Processor.class) 
// 指定需要处理的注解
@SupportedAnnotationTypes("com.example.annotation.BindView") 
// 指定Java JDK编译版本
@SupportedSourceVersion(SourceVersion.RELEASE_7)
```
关于文件生成可以使用JavaPoet，
相关地址：
[Github](https://github.com/square/javapoet) 

---

##### apt-library 工具类

```
import android.app.Activity;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

/**
 * 自定义牛油刀工具类
 * @author XGY
 */
public final class ButterKnife {

    /** 绑定 */
    public static void bind(Activity target) {
        try {
            // Activit类
            Class clazz = target.getClass();
            // 反射获取apt生成的指定类
            Class<?> bindViewClass = Class.forName(clazz.getName() + "$$ViewBinding");
            // 获取它的方法
            Method method = bindViewClass.getMethod("bind", clazz);
            // 执行方法
            method.invoke(bindViewClass.newInstance(), target);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}
```

---

##### app 中使用

```
public class MainActivity extends AppCompatActivity {
    
    // 使用自定义注解
    @BindView(R.id.textView)
    TextView textView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // 绑定
        ButterKnife.bind(this);
        // 设置文案
        textView.setText("ButterKnife 绑定成功");
    }
}
```

