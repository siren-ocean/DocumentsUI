## DocumentsUI from android-12.1.0_r11
### DocumentsUI脱离源码在Android Studio的编译

### 支持说明
* 不试图改变项目本身的目录结构
* 通过添加额外的配置和依赖构建Gradle环境支持
* 修复以下的代码使项目能运行起来 (如下)


```
@PATH: src/com/android/documentsui/DirectoryLoader.java
*********************************************************
//已经弃用的方法
//@Override
protected Executor getExecutor() {
    return ProviderExecutor.forAuthority(mRoot.authority);
}
```

## 使用命令编译
### 环境依赖
*  Gradle 7.3.3
*  JDK version 11

```
# 构建环境
gradle wrapper

# 打包编译
 ./gradlew assemble
```



## 使用Android Studio编译

### 执行Android Studio上Build APK的操作, 然后将apk推送到设备上DocumentsUI所在的目录

```
adb push DocumentsUI.apk /system/priv-app/DocumentsUI/

adb shell killall com.android.documentsui
```
######  首次推送会起不来，需要重启一下设备
```
adb reboot
```


## 构建步骤

### Step1：引入静态依赖
##### @framework.jar:
```
// android-12/out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/classes-header.jar
compileOnly files('libs/framework.jar')
```
![avatar](images/framework.png)


### Step2：引入代码
###### 导入一个特殊类DocumentsStatsLog，该类是自动生成的，可在out目录找到
```
// android-12/out/soong/.intermediates/packages/apps/DocumentsUI/statslog-docsui-java-gen

sourceSets {
    main {
        java.srcDirs = ['src', 'statslog-docsui-java-gen/gen']
        res.srcDirs = ['res']
        manifest.srcFile 'AndroidManifest.xml'
    }
}
```


## 生成platform.keystore默认签名

在 android-12/build/target/product/security路径下找到签名证书，并使用 [keytool-importkeypair](https://github.com/getfatday/keytool-importkeypair) 生成keystore,
执行如下命令：  

```
./keytool-importkeypair -k platform.keystore -p 123456 -pk8 platform.pk8 -cert platform.x509.pem -alias platform
```

并将以下代码添加到gradle配置中：

```
    signingConfigs {
        platform {
            storeFile file("platform.keystore")
            storePassword '123456'
            keyAlias 'platform'
            keyPassword '123456'
        }
    }

    buildTypes {
        release {
            debuggable false
            minifyEnabled false
            signingConfig signingConfigs.platform
        }

        debug {
            debuggable true
            minifyEnabled false
            signingConfig signingConfigs.platform
        }
    }
```

---

### 关联项目
* [Settings](https://github.com/siren-ocean/Settings)
* [SystemUI](https://github.com/siren-ocean/SystemUI)
* [Launcher3](https://github.com/siren-ocean/Launcher3)
* [Camera2](https://github.com/siren-ocean/Camera2)
* [PermissionController](https://github.com/siren-ocean/PermissionController)
