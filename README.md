# 技术随笔
## 目录
* [1.Config.gradle文件中的数字参数在代码里获得](#1.Config.gradle文件中的数字参数在代码里获得)
* [方案一](#方案一：通过buildConfigField构建BuildConfig)
* [方案二](#方案二：通过Application中添加meta指定android:value)

## 1.Config.gradle文件中的数字参数在代码里获得
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
