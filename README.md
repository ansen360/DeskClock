> DeskClock源码路径:/repo/packages/apps/DeskClock/
### 导入DeskClock源码到AndroidStudio
- **1.拷贝源码到AS工程**

使用AndroidStudio创建DeskClock工程,指定Android源码相同的包名

把DeskClock中src、res、AndroidManifest.xml、assets复制到创建的AS工程中

查看.mk文件发现依赖datetimepicker工程库,在AS工程中创建一个Module,module类型为AndroidLibrary,同理把对应的源码复制到该module中.效果图如下:
> datetimepicker源码路径:/repo/frameworks/opt/datetimepicker/

![](http://oma689k8f.bkt.clouddn.com/note/5/1.png)

- **2.导入framework.jar系统库**

将repo/out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/classes.jar复制出来,重命名为framework.jar;

为了区分系统的jar包,在如上图中app目录下创建ext_libs目录,将framework.jar复制到该目录中;

在**build.gradle** dependencies{}中添加provided files('ext_libs/framework.jar')编译该库
> compile的方式是编译使用,并且打包进apk中

> provided的方式是编译使用,但不打包进apk中

```
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    testCompile 'junit:junit:4.12'
    compile project(':datetimepicker')
    compile 'com.android.support:support-v13:26.0.0-alpha1'
    compile 'com.android.support:design:26.0.0-alpha1'
    compile 'com.android.support:gridlayout-v7:26.0.0-alpha1'
    provided files('ext_libs/framework.jar')
}
```
![](http://oma689k8f.bkt.clouddn.com/note/5/2.png)
**最后修改project的build.gradle**

在allprojects中添加以下代码:
```
allprojects {
    repositories {
        jcenter()
    }
    gradle.projectsEvaluated {
        tasks.withType(JavaCompile) {
            options.compilerArgs.add('-Xbootclasspath/p:app\\ext_libs\\framework.jar')
        }
    }

}
```
Xbootclasspath/p:path   --  让jvm优先于默认的bootstrap去加载path中指定的class
- **3.导入其他的依赖库**

datetimepicker中添加

    compile 'com.android.support:appcompat-v7:26.0.0-alpha1'

DeskClock中添加

    compile project(':datetimepicker')
    compile 'com.android.support:support-v13:26.0.0-alpha1'
    compile 'com.android.support:design:26.0.0-alpha1'
    compile 'com.android.support:gridlayout-v7:26.0.0-alpha1'
    
解决所有报错,成功编译出apk
- **4.配置系统签名**


    对于Eclipse或AndroidStudio已经编出来的apk手动重新签名:
    创建SignApk文件夹,把需要重新签名的apk和platform.x509.pem、platform.pk8、signapk.jar文件拷贝到该目录下
    对应文件路径：
    platform.x509.pem、platform.pk8：build/target/product/security
    signapk.jar：out/host/linux-x86/framework
    signapk源码路径：build/tools/signapk
    执行以下命令生成系统签名后的apk
    java -jar signapk.jar platform.x509.pem platform.pk8 old.apk new.apk
    
**使用AndroidStudio配置系统签名:**

1).生成签名文件
在AS工具类上选择Build -->> Generate Signed APK,如果没有就Create new 创建生成一个SignXXX.jks签名文件
![](http://oma689k8f.bkt.clouddn.com/note/5/3.png)

在DeskClock工程目录下创建SignApk文件夹保存签名文件

2.)使用keytool-importkeypair对生成的SignXXX.jks签名文件引入系统签名

[keytool-importkeypair下载](https://github.com/getfatday/keytool-importkeypair)

platform.x509.pem、platform.pk8路径:build/target/product/security

把keytool-importkeypair、platform.x509.pem、platform.pk8和SignXXX.jks签名文放在同一目录下,执行以下命令

> ./keytool-importkeypair -k [jks文件名] -p [jks的密码] -pk8 platform.pk8 -cert platform.x509.pem -alias [jks的别名]

> ./keytool-importkeypair -k SignXXX.jks -p 123456 -pk8 platform.pk8 -cert platform.x509.pem -alias SignXXX

![](http://oma689k8f.bkt.clouddn.com/note/5/4.png)

执行完后(**需要在Linux下执行**),我们就得到系统签名SignXXX.jks,替换SignApk中的签名文件

3.)配置build.gradle编译时直接生成签名的apk

```
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"
    defaultConfig {
        applicationId "com.android.deskclock70"
        minSdkVersion 21
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    signingConfigs {
        release {
            storeFile file("../SignApk/SignXXX.jks")
            storePassword '123456'
            keyAlias 'SignXXX'
            keyPassword '123456'
        }

        debug {
            storeFile file("../SignApk/SignXXX.jks")
            storePassword '123456'
            keyAlias 'SignXXX'
            keyPassword '123456'
        }
    }

}
```

直接运行便可生成配置系统签名的apk