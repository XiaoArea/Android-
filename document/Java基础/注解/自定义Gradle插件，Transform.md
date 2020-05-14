# Transform
Gradle Transform是Android官方提供给开发者在项目构建阶段（.class -> .dex转换期间）用来修改.class文件的一套标准API，即把输入的.class文件转变成目标字节码文件。目前比较经典的应用是字节码插桩、代码注入等。

build一个项目，会打印出如下日志，红框框住的部分就是一个Transform的名称

![Image text](https://note.youdao.com/yws/public/resource/9859503e734f7a636c7187415ddd705f/xmlnote/3D635DD346714350A9DD9E0C53267746/3519)

通过上张图可以看到原生就带了一系列Transform供使用，那么这些Transform是怎么组织在一起的呢？

每个Transform其实都是一个gradle task，Android编译器中的TaskManager将每个Transform串连起来，第一个Transform接收来自javac编译的结果，以及已经拉取到在本地的第三方依赖（jar、aar），还有resource资源，注意，这里的resource并非android项目中的res资源，而是asset目录下的资源。 这些编译的中间产物，在Transform组成的链条上流动，每个Transform节点可以对class进行处理再传递给下一个Transform。常见的混淆，Desugar等逻辑，它们的实现如今都是封装在一个个Transform中，而我们自定义的Transform，会插入到这个Transform链条的最前面。

最终定义的Transform会被转化成一个个TransformTask，在Gradle编译时调用。

---

#### Transform 两个基础概念
1. **TransformInput** 是指输入文件的一个抽象包括：
    - DitectoryInput 集合是指以源码的方式参与项目编译的所有目录结构及其目录下的源码文件
    - JarInput 集合是指以jar包方式参与项目编译的所有本地jar包和远程jar包（此处的jar包包括aar）
2. **TransformOutputProvider** 是指Transform的输出，通过它可以获取到输出路径等信息

#### Transform 类
##### 定义：

```
public abstract class Transform {
    public Transform() {
    }
    // Transform名称
    public abstract String getName();
    public abstract Set<ContentType> getInputTypes();
    public Set<ContentType> getOutputTypes() {
        return this.getInputTypes();
    }
    public abstract Set<? super Scope> getScopes();
    public abstract boolean isIncremental();
    /** @deprecated */
    @Deprecated
    public void transform(Context context, Collection<TransformInput> inputs, Collection<TransformInput> referencedInputs, TransformOutputProvider outputProvider, boolean isIncremental) throws IOException, TransformException, InterruptedException {
    }
    public void transform(TransformInvocation transformInvocation) throws TransformException, InterruptedException, IOException {
        this.transform(transformInvocation.getContext(), transformInvocation.getInputs(), transformInvocation.getReferencedInputs(), transformInvocation.getOutputProvider(), transformInvocation.isIncremental());
    }
    public boolean isCacheable() {
        return false;
    }
    ...
}
```

##### 说明：
###### getName() ：
Transform名称，上面build日志红框框住的部分就是Transform名称：
> transformClassesWithDexBuilderForDebug

在gradle plugin的源码中有一个叫TransformManager的类，这个类管理着所有的Transform的子类，里面有一个方法叫getTaskNamePrefix，在这个方法中就是获得Task的前缀，以transform开头，之后拼接ContentType，这个ContentType代表着这个Transform的输入文件的类型，类型主要有两种，一种是Classes，另一种是Resources，ContentType之间使用And连接，拼接完成后加上With，之后紧跟的就是这个Transform的Name，name在getName()方法中重写返回即可。TransformManager#getTaskNamePrefix()代码如下：

```
static String getTaskNamePrefix(Transform transform) {
    StringBuilder sb = new StringBuilder(100);
    sb.append("transform");
    sb.append((String)transform.getInputTypes().stream().map((inputType) -> {
        return CaseFormat.UPPER_UNDERSCORE.to(CaseFormat.UPPER_CAMEL, inputType.name());
    }).sorted().collect(Collectors.joining("And")));
    sb.append("With");
    StringHelper.appendCapitalized(sb, transform.getName());
    sb.append("For");
    return sb.toString();
}
    
```


###### getInputTypes() ：
需要处理的数据类型，有两种枚举类型
- CLASSES 代表处理的 java 的 class 文件，返回TransformManager.CONTENT_CLASS
- RESOURCES 代表要处理 java 的资源，返回TransformManager.CONTENT_RESOURCES

###### getScopes() ：
指 Transform 要操作内容的范围，官方文档 Scope 有 7 种类型：

1. EXTERNAL_LIBRARIES ： 只有外部库
2. PROJECT ： 只有项目内容
3. PROJECT_LOCAL_DEPS ： 只有项目的本地依赖(本地jar)
4. PROVIDED_ONLY ： 只提供本地或远程依赖项
5. SUB_PROJECTS ： 只有子项目
6. SUB_PROJECTS_LOCAL_DEPS： 只有子项目的本地依赖项(本地jar)
7. TESTED_CODE ：由当前变量(包括依赖项)测试的代码

如果要处理所有的class字节码，返回TransformManager.SCOPE_FULL_PROJECT

###### isIncremental() ：
增量编译开关，当开启增量编译的时候，相当input包含了changed/removed/added三种状态，实际上还有notchanged。需要做的操作如下：
- NOTCHANGED : 当前文件不需处理，甚至复制操作都不用；
- ADDED、CHANGED : 正常处理，输出给下一个任务；
- REMOVED : 移除outputProvider获取路径对应的文件。

###### transform() ：
注意点：
- 如果拿取了getInputs()的输入进行消费，则transform后必须再输出给下一级
- 如果拿取了getReferencedInputs()的输入，则不应该被transform
- 是否增量编译要以transformInvocation.isIncremental()为准

###### isCacheable() ：
如果transform需要被缓存，则为true，它被TransformTask所用到

---

# 自定义Gradle插件
Gradle插件打包了可重用的构建逻辑，可以在不同的项目中使用。Gradle提供了几种方式来让你实现自定义插件，这样你可以重用你的构建逻辑，甚至提供给他人使用。

可以使用多种语言来实现Gradle插件，其实只要最终被编译为JVM字节码的都可以，常用的有Groovy、Java、Kotlin。通常，使用Java或Kotlin（静态类型）实现的插件比使用Groovy实施的插件性能更好。

##### 打包Gradle插件的3种方式，如下：

## Build script
在build.gradle构建脚本中直接使用，只能在本文件内使用；

```
// build.gradle 中配置
class GreetingPlugin implements Plugin<Project> {
    void apply(Project project) {
        project.task('hello') {
            doLast {
                println 'Hello from the GreetingPlugin'
            }
        }
    }
}
apply plugin: GreetingPlugin
```
按照官方文档的做法，build.gradle 内引入上面的代码，会发现 Plugin 和 Project 2个类是无法被引入的。而且这种方案有个弊端，只能在构建脚本文件内部使用，这样就没办法提供给其他module或者project使用了。这种方案基本不会在真实项目中使用

> 注意：对于Gradle而言，每一个Module都是一个项目

## buildSrc project
新建一个名为buildSrc的Module使用，只能在本项目中使用；(这里使用Groovy语法)
1. 创建好项目之后，新建一个名称为buildSrc的Module，项目类型任意，只保留build.gradle文件和src/main目录，其余文件全部删掉。注意：名字一定要是buildSrc，否则应用插件的时候会找不到插件。修改后的目录如下：

    ![Image text](https://note.youdao.com/yws/public/resource/9859503e734f7a636c7187415ddd705f/xmlnote/1790F74DD0E14CA28D383001BD9618BA/3623)
    > ==踩过的坑==：创建buildSrc这个Module的时候，如果选择了Android Library类型会有Plugin with id 'com.android.library' not found.的异常，这是因为buildSrc是Android的保留名称，只能作为plugin插件使用，后面修改buildSrc的build.gradle文件后就不报错了。如果选择Java Library类型，好像就没有这个异常，而且这个类型的文件少一些，建议选择Java Library类型。

2. 修改Gradle文件内容：

    ```
    // 如果需要使用kotlin必需先引入Kotlin
    buildscript {
        // kotlin版本
        ext.kotlin_version = '1.3.50'
        repositories {
            mavenCentral() //必须
        }
        dependencies {
            // 引入kotlin插件
            classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        }
    }
    apply plugin: 'kotlin' // kotlin 必须
    
    apply plugin: 'groovy'  // 必须
    apply plugin: 'maven'
    // 设置依赖
    dependencies {
        implementation gradleApi() // 必须
        implementation localGroovy() // 必须
        // 如果要使用android的API，需要引用这个，实现Transform的时候会用到
        implementation 'com.android.tools.build:gradle:3.5.0'
    }
    // 设置仓库
    repositories {
        google()
        jcenter()
        mavenCentral() //必须
    }
    // 指定编码格式，防止乱码
    tasks.withType(JavaCompile) {
        options.encoding = "UTF-8"
    }
    ```
    ==注意==：
    > 1、如果引入了com.android.tools.build:gradle:3.5.0，需要加入google()仓库，只引入 jcenter()和 mavenCentral()仓库中，会提示找不到。
    
    > 2、如果使用Kotlin的方式需要在配置的上方导入Kotlin

3. 在main下新建groovy目录，在groovy目录下创建包名目录，在包名目录下新建一个groovy文件，并且实现 ==org.gradle.api.Plugin== 接口，注意文件名需要以 ==.groovy== 结尾。

4. 在main下新建resources目录，在resources目录下新建META-INF目录，再在META-INF下新建gradle-plugins目录,在gradle-plugins目录下新建properties文件，比如com.example.plugin.properties,注意：这个文件命名是没有要求的，但是要以 ==.properties== 结尾。

    buildSrc项目目录如下：

    ![Image text](https://note.youdao.com/yws/public/resource/9859503e734f7a636c7187415ddd705f/xmlnote/EB2FD44B77CD490FB1BAEBA5D1A18E57/3659)

    TestPlugin.groovy源码如下：
    ```
        package com.example.plugin.groovy
        
        import org.gradle.api.Plugin
        import org.gradle.api.Project
        
        class TestPlugin implements Plugin<Project> {
        
            @Override
            void apply(Project project) {
                println("groovy 方式 TestPlugin")
            }
        }
    ```
    > 注意：这里是groovy、java、kotlin写，他们都是基于JVM的。Plugin和Project是Gradle的API，所以需要先在脚本文件中配置好了再写插件实现类，否则是找不到这2个类的。
    
    
    META-INF/gradle-plugins/xx.properties 配置如下：
    ```
    // com.example.plugin.groovy.properties
    implementation-class=com.example.plugin.groovy.TestPlugin
    
    // com.example.plugin.java.properties
    implementation-class=com.example.plugin.java.TestPlugin
    
    // com.example.plugin.kotlin.properties
    implementation-class=com.example.plugin.kotlin.TestPlugin
    ```
5. 在要使用插件的Module中应用，比如在app的build.gradle中，引用插件如下：
```
apply plugin: 'com.example.plugin.groovy'
apply plugin: 'com.example.plugin.java'
apply plugin: 'com.example.plugin.kotlin' 
```
> ==注意==：这里引用的插件名称就是properties文件的名称。接下来如果编译正常的话，就会看见插件实现类的apply()方法中打印的日志。
    
![Image text](https://note.youdao.com/yws/public/resource/9859503e734f7a636c7187415ddd705f/xmlnote/6170142A11B840588BA66827EC6E9EC0/3687)

##### 小结：
- module 名称只能为 buildSrc
- buildSrc project 下的插件是自动加载

## Standalone project
在独立的Module中使用，可以发布到本地或者远程仓库供其他项目使用。

##### 步骤如下：
1、新建一个Module，项目类型任意，名字任意，也是只保留build.gradle文件和src/main目录，其余文件全部删掉。

2、修改Gradle文件内容：
```
apply plugin: 'groovy'  // 必须
apply plugin: 'maven'  // 要想发布到Maven，此插件必须使用

dependencies {
    implementation gradleApi() // 必须
    implementation localGroovy() // 必须
}
repositories {
    mavenCentral() // 必须
}
// 指定编码，Java方式容易有乱码现象
tasks.withType(JavaCompile) {
    options.encoding = "UTF-8"
}

def group = 'com.example.myplugin' // 组
def version = '1.0.0' // 版本
def artifactId = 'myPlugin' // 唯一标示

// 将插件打包上传到本地maven仓库
uploadArchives {
    repositories {
        mavenDeployer {
            pom.groupId = group
            pom.artifactId = artifactId
            pom.version = version
            // 指定本地maven的路径，在项目根目录下, 位置及目录自由定义
            repository(url: uri('../plugin'))
        }
    }
}

```
相比buildSrc方案，增加了Maven的支持和uploadArchives这样一个Task，这个Task的作用是为了将插件打包上传到本地maven仓库。注意打包文件目录是../plugin，它表示的是项目根目录下，这里用了2个.，1个.表示当前module根目录，2个.表示project的根目录。

3、src/main目录下的插件实现类和properties文件与buildSrc方案是一致的。

![Image text](https://note.youdao.com/yws/public/resource/9859503e734f7a636c7187415ddd705f/xmlnote/WEBRESOURCE0e2e678f68b6ef0fe2ea90eed172518c/3719)

4、在终端中执行gradle uploadArchives指令，或者展开AS右侧的Gradle，找到对应module下的 uploadArchivesTask，就可以将插件部署到项目根目录的 plugin 目录下。

![Image text](https://note.youdao.com/yws/public/resource/9859503e734f7a636c7187415ddd705f/xmlnote/WEBRESOURCE668dac4b084e623c0e0ca2947dea2f45/3715)

插件部署到本地后的目录如下：
这个repos就是你本地的Maven仓库，com/example/myplugin是脚本中的group指定的，myPlugin表示模块名称，是一个唯一标示，1.0.0由version指定

![Image text](https://note.youdao.com/yws/public/resource/9859503e734f7a636c7187415ddd705f/xmlnote/WEBRESOURCEf1e48c15b20202eece9cbfe2f3dee297/3732)

5、引用插件
在buildSrc中，系统自动帮开发者自定义的插件提供了引用支持，但完全自定义Module的插件中，开发者就需要自己来添加自定义插件的引用支持。

在project的build.gradle文件中，添加如下脚本：
```
buildscript {

    repositories {
        google()
        jcenter()

        // 指定本地maven的路径，在项目根目录下
        maven { url uri('./plugin') }
    }
    
    dependencies {
        classpath 'com.android.tools.build:gradle:3.5.0'
        // 引入自定义插件
        classpath 'com.example.myplugin:myPlugin:1.0.0'
    }
}
```
注意：
- 这里的uri只用了1个.，因为project的build.gradle文件已经在项目根目录下了
- classpath指定的路径格式如下：
    ```
    // 这3个参数是在build.gradle脚本文件中申明的
    classpath '[groupId]:[artifactId]:[version]' 
    ```
配置完毕后，在需要使用的Module引用插件，如在app的build.gradle文件中：
```
// 引用自定义插件，properties文件名称的方式
apply plugin: 'myPlugin'
```
###### 小结
- 方案3用完全自定义Module实现自定义插件，这里是打包上传插件到本地Maven，当然也可以发布到远程仓库，可参考：[gradle插件上传Jcenter与自建Maven私服](https://blog.csdn.net/pf_1308108803/article/details/78119591)
- 上传本地Maven需要注意脚本文件中uploadArchives配置，以及引用插件时地址的配置，一般放在project根目录下就可以
- 如果重新修改了插件代码，需要重新部署uploadArchives(即：版本迭代)才能在别的地方引用插件

##### 总结
虽然官方提供了3种方案，但实际开发中只会用到后2种，有一个比较方便的做法，开发调试的时候用buildSrc project模式，等后面需要把插件共享出去等时候，再把它改为第三种方案即可。


---

#### 参考：
- [Gradle 插件官方文档地址](https://docs.gradle.org/current/userguide/custom_plugins.html)
- [Android 自定义Gradle插件的3种方式](https://www.jianshu.com/p/f902b51e242b)
- [Android Gradle Transform 详解](https://www.jianshu.com/p/cf90c557b866)
