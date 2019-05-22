# 技术随笔
## 目录
* [Config.gradle文件中的数值参数在代码里获得](#1configgradle文件中的数值参数在代码里获得)
* [List在遍历时出现ConcurrentModificationException](#2List在遍历时出现ConcurrentModificationException)
* [build.gradle文件里自定义属性](#3buildgradle文件里自定义属性)
* [图片Bitmap的缩放](#4图片Bitmap的缩放)
* [proto编译支持javanano](#5图片Bitmap的缩放)

## 1.Config.gradle文件中的数值参数在代码里获得
背景：config.gradle文件标注了sdk或者app的version，需要在代码里获取
### 方案一：通过buildConfigField构建BuildConfig
1.在project里build.gradle里的defaultConfig添加buildConfigField
```java
buildConfigField 'String','VERSION',"\"${rootProject.ext.sdkversion}\""
```
> 注意：rootProject.ext.sdkversion是string类型，但是一定要加上```\"\"```转义符号，否则生成的Field不是string类型

2.在BuildConfig中找到相应的Field
```java
  public static final int VERSION_CODE = 1;
  public static final String VERSION_NAME = "1.0";
  // Fields from default config.
  public static final String VERSION = "2.1.1";
```

### 方案二：通过Application中添加meta指定android:value
1.在Manifest.xml文件中添加meta
```java
<meta-data 
    android:name="version" android:value="${sdkversion}"/>
```
2.在build.gradle里添加
```java
 manifestPlaceholders = [
    sdkversion: "$rootProject.ext.sdkversion"
 ]
```
> 多个值时用,分隔开

3.在代码里获得
```java
   try {
        ApplicationInfo appInfo = getPackageManager()
                .getApplicationInfo(getPackageName(),PackageManager.GET_META_DATA);
        sdkversion = appInfo.metaData.getString("sdkversion");
        Log.e("sdkversion", "sdkversion=" + sdkversion);
    } catch (PackageManager.NameNotFoundException e) {
        e.printStackTrace();
    }
```

## 2.List在遍历时出现ConcurrentModificationException
### 方案一：通过替换成特殊的ArrayList
```java
List<String> collects = new CopyOnWriteArrayList<String>();
```

### 方案二：修改List遍历方式
```java
Iterator iterator  = collects.iterator();  
while(iterator.hasNext()){
    System.out.println(iterator.next());
    collects.add("333");
    System.out.println("add over");
}
```

## 3.build.gradle文件里自定义属性
在App工程中统一配置版本参数和dependencies的参数，需要一个统一config.gradle管理和配置<br>
1.新建一个config.gradle文件
```java
ext {
    sdkversion = "1.0.0"
    packaging = "aar"

    android = [
            compileSdkVersion: 28,
            buildToolsVersion: "28.0.3",
            applicationId    : "com.xxx.xxx",
            minSdkVersion    : 15,
            targetSdkVersion : 28
    ]
}
```
2.在Project根目录的build.gradle文件添加这文件
```java
apply from: "config.gradle"
```
3.在Module的gradle文件或manifest文件中使用
```java
android {
    compileSdkVersion rootProject.ext.android.compileSdkVersion
    buildToolsVersion rootProject.ext.android.buildToolsVersion

    defaultConfig {
        minSdkVersion rootProject.ext.android.minSdkVersion
        targetSdkVersion rootProject.ext.android.compileSdkVersion
        versionCode 1
        versionName rootProject.ext.sdkversion

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

    }
}
```

## 4.图片Bitmap的缩放
使用官方的ThumbnailUtils.extractThumbnail功能类有可能出现图片显示不全的问题，所以还是使用Matrix
```java
    int bitWidth = bitmap.getWidth();
    int bitHeight = bitmap.getHeight();

    float scaleWidth = ((float) mNewWidth) / bitWidth;
    float scaleHeight = ((float) mNewHeight) / bitHeight;

    Matrix matrix = new Matrix();
    matrix.postScale(scaleWidth, scaleHeight);

    Bitmap newBitmap = Bitmap.createBitmap(bitmap, 0, 0, bitWidth, bitHeight, matrix,
            true);
    if (null != newBitmap) {
        bitmap.recycle();
        mBitmap = newBitmap;
    }
```

## 5.proto编译时在高版本支持javanano，pom中的依赖
```java
<dependencies>
    <dependency>
      <groupId>com.android.support</groupId>
      <artifactId>appcompat-v7</artifactId>
      <version>28.0.0</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>com.google.protobuf.nano</groupId>
      <artifactId>protobuf-javanano</artifactId>
      <version>3.1.0</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>com.google.protobuf</groupId>
      <artifactId>protoc</artifactId>
      <version>3.1.0</version>
      <scope>runtime</scope>
    </dependency>
  </dependencies>
```

