# 县市报友盟助手1.0.0
演示demo下载：
<div align=center><img src='https://www.pgyer.com/app/qrcode/TSEK'/></div>
---
支持三大功能：
    
    * 分享  
      
      QQ好友、QQ空间、微信好友、微信朋友圈、新浪微博、钉钉、复制链接  
      
    * 第三方登录  
      
      QQ、微信、新浪微博
      
    * 统计  
      
      应用时长、页面跳转、自定义事件（计数事件、计算事件）
+++
## 一.集成配置
### 1.1 Project build.gradle
添加县市报的私有Maven仓库路径：
```groovy
buildscript {
    repositories {
    
        // 添加县市报本地Maven仓库
        maven { url "http://10.100.62.98:8086/nexus/content/groups/public" }
    }
}
allprojects {
    repositories {
        
        // 添加县市报本地Maven仓库
        maven { url "http://10.100.62.98:8086/nexus/content/groups/public" }       
    }
}
```
+++
### 1.2 App build.gradle
minSdkVersion不得小于15
```groovy
android {
    compileSdkVersion 26
    defaultConfig {
        
        // 最小支持sdk=15
        minSdkVersion 15
        
        manifestPlaceholders = [
                            "UMENG_APP_KEY": UMENG_APP_KEY,     //友盟App Key
                            "QQ_APP_ID"    : QQ_APP_ID,         //QQ AppId
                            "_packagename" : _packagename       //主项目包名
        ]
    }
}
dependencies {
    
    // 县市报友盟助手
    implementation 'com.xsb.umeng:xsb-umeng-tools:1.0.2-SNAPSHOT'.
}
configurations.all {
    // 针对使用SNAPSHOT，每次不更新的问题。当有新的SNAPSHOT提交，只需要更改数字后同步即可
    resolutionStrategy.cacheChangingModulesFor 4, 'seconds'
}
```
+++
### 1.3 添加回调Activity
微信和钉钉需要在app包下新建各自的回调Activity：

* 微信路径：

  your_package_name.wxapi.WXEntryActivity，并且继承WXCallbackActivity
```
/**
 * 微信回调
 */
public class WXEntryActivity extends WXCallbackActivity {
}
```
+++
* 钉钉路径：

  your_package_name.ddshare.DDShareActivity，并且继承DingCallBack
```
/**
 * 钉钉回调
 */
public class DDShareActivity extends DingCallBack {
}
```
+++
### 1.4 配置各平台账号 gradle.properties
需要在项目的gradle.properties文件内填写以下所需的账号信息，填充ManifestPlaceHolder：
```properties
# App在友盟注册后的App Key
UMENG_APP_KEY=5b57d0b9a40fa379c4000118
  
# App在QQ开放平台注册后的App ID
QQ_APP_ID=1107128079
  
# App的包名
_packagename = com.zjonline.xsb_tools.umeng.demo
```
+++
### 1.5 App AndroidManifest.xml
友盟相关的配置已经集成在lib aar的AndroidManifest文件中，app打包时会自动合并清单
+++
### 1.6 App 混淆
友盟的免混淆配置已经保存在aar中，app混淆打包时会自动合并混淆配置清单
---
## 二、初始化
Application中初始化：
```
UMengTools.create(applicationContext)
                /**
                 * 是否开启调试模式（开启后有debug日志输出，同时友盟后台开启集成测试）
                 * 开发阶段必须开启集成测试功能，防止友盟统计数据污染正式环境！！！
                 * 上线发布前务必再将此开关设置为false！！！
                 * 默认false
                 */
                .setLogEnabled(BuildConfig.DEBUG)
                
                 // 是否加密友盟数据传输，保证数据安全性，默认false
                .setEncryptEnabled(true)
                
                 // 是否每次第三方登录都弹出用户授权页面，默认false
                .setNeedAuthOn3rdLogin(true)
                
                 // 设置QQ开放平台注册的app id和app secret
                .setQQZone("QQ App ID", "QQ App Key")
                
                 // 设置微信开放平台注册的app id和app secret
                .setWeixin("WX App ID", "WX App Secret")
                
                 // 设置新浪微博开放平台注册的app id和app secret和重定向url
                .setSinaWeibo("Sina App ID", "Sina App Secret", "Sina Redirect Url")
                
                 // 设置钉钉开放平台注册的app id
                .setDingDing("DingTalk App ID")
                
                /** 
                 * 设置向友盟申请的app secretKey（需要企业认证），防止统计数据被盗刷。
                 * 默认不设置，没有二次安全验证，数据可能被盗刷
                 */
                .setAnalyticsSecret("UMeng App Secret")
                
                 // 按app自有账号系统进行统计
                .setAnalyticsWithUser("10035")
                
                 // 按第三方账号系统进行统计
                .setAnalyticsWithUser("QQ", "40745050")
                
                 /**
                  * 是否启用默认的页面自动统计功能
                  * 建议禁用，采用手动统计Activity，fragment,及其他View的数据
                  * 默认true
                  */
                .setAnalyticsAutoTrack(false)
                
                 /**
                  * 设置统计功能的session超时时间间隔,单位ms
                  * 例如：当一个onResume方法与上一个Activity的onPause方法相差30秒，标志新Session的开始
                  * 默认30000
                  */
                .setAnalyticsSessionTimeout(30 * 1000)
                
                 // 设置统计场景的类型，NORMAL:普通app，GAME:游戏app，默认NORMAL
                .setAnalyticsType(AnalyticsType.NORMAL)
                
                 // 是否启用错误日志的抓取上报，默认true
                .setAnalyticsWithExceptions(true);
```
---
## 三、分享
目前lib支持的分享平台为：
![avatar](http://10.100.62.91/xsb/XSBUmeng/raw/aaacacf14a9aad52f8daee6ed4123922fe60e260/platform.png)
+++
### 3.1 权限
友盟分享需要以下两项系统权限，需要app自行实现权限检测和用户交互：

    1.  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    2.  <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
+++
### 3.2 类型
目前lib支持三种类型的社交内容分享：
* 纯文本 （QQ好友除外）

    ```
    new UMengShare.TextBuilder(activity)
                        .text(message)                          // 文本内容
                        .platform(platformType)                 // 目标平台
                        .callback(mUMengShareSimpleListener)    // 分享回调
                        .share();
    ```
+++
* 图片

    ```
    new UMengShare.ImageBuilder(activity)
                        .image(imageUrl)                        // 网络图片
                        .image(imageDrawable)                   // 图片资源
                        .image(imageFile)                       // 本地文件
                        .image(bitmap)                          // Bitmap
                        .image(imageBytes)                      // 字节流
                        .compress(ImageCompressStyle.SCALE)     // 压缩方式，SCALE：压缩尺寸，QUALITY：压缩质量，默认：SCALE
                        .format(Bitmap.CompressFormat.JPEG)     // 压缩格式，PNG/JPEG/WEBP，默认：JPEG
                        .platform(platformType)                 // 目标平台
                        .callback(mUMengShareSimpleListener)    // 分享回调
                        .share();
    ```
+++
* 链接

    ```
    new UMengShare.LinkBuilder(activity)
                        .url(linkUrl)                           // 链接地址
                        .title(linkTitle)                       // 链接标题
                        .description(linkDescription)           // 链接描述
                        .image(imageUrl)                        // 网络图片
                        .image(imageDrawable)                   // 图片资源
                        .image(imageFile)                       // 本地文件
                        .image(bitmap)                          // Bitmap
                        .image(imageBytes)                      // 字节流
                        .compress(ImageCompressStyle.SCALE)     // 压缩方式，SCALE：压缩尺寸，QUALITY：压缩质量，默认：SCALE
                        .format(Bitmap.CompressFormat.JPEG)     // 压缩格式，PNG/JPEG/WEBP，默认：JPEG
                        .platform(platformType)                 // 目标平台
                        .callback(mUMengShareSimpleListener)    // 分享回调
                        .share();
    ```
+++
### 3.3 回调
```java
/**
 * 分享回调精简
 */
public abstract class UMengShareSimpleListener implements UMengShareListener {
    
    /** 
     * 分享开始
     * @param platformType 目标平台
     */
    @Override
    public void onStart(PlatformType platformType) {
    }
    
    /** 
     * 分享成功
     * @param platformType 目标平台
     */
    @Override
    public void onResult(PlatformType platformType) {
    }
    
    /** 
     * 分享失败
     * @param platformType 目标平台
     * @param errorMsg     错误信息
     */
    @Override
    public void onError(PlatformType platformType, String errorMsg) {
    }
    
    /** 
     * 分享取消
     * @param platformType 目标平台
     */
    @Override
    public void onCancel(PlatformType platformType) {
    }
    
    /** 
     * 复制链接
     * @param copied       是否已复制
     * @param url          要复制的链接地址
     * @param msg          提示信息
     */
    @Override
    public void onCopyLink(boolean copied, String url, String msg) {
    }
    
    /** 
     * 设备不支持该平台
     * @param platformType 目标平台
     * @param msg          提示信息
     */@Override
    public void onNotSupported(PlatformType platformType, String msg) {
    }
    
    /** 
     * 权限被拒绝
     * @param platformType 目标平台
     * @param permissions  被拒绝的权限列表
     */
    @Override
    public void onPermissionDenied(PlatformType platformType, String... permissions) {
    }
    
}
```
+++
### 3.4 onActivityResult
QQ和新浪微博分享：
```
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
        
    // 执行回调传递
    UMengTools.onActivityResult(this, requestCode, resultCode, data);
}
```
+++
### 3.5 销毁
分享对象释放
```
@Override
protected void onDestroy() {
    // 释放友盟对象
    UMengTools.release(this);
        
    super.onDestroy();
}
```
---
## 四、第三方登录
目前支持以下三个平台：
* QQ
  
* 微信
  
   需开通微信开放平台开发者资质，需要企业认证及缴年费，demo使用的账号暂时无法完成微信授权登录）
* 新浪微博
  
   需要在新浪微博开放平台上注册app，并已上线才能通过审核，demo暂时未上线应用市场，无法完成微博的授权登录）
+++ 
### 4.1 登录授权
```
new UMengOAuth(activity)
            .to(platformType)                               // 目标平台
            .callback(mUMengOAuthListener)                  // 授权回调          
            .login();
```
+++ 
### 4.2 授权回调
```java
/**
 * 第三方登录授权回调
 */
public abstract class UMengOAuthSimpleListener implements UMengOAuthListener {
    
    /** 
     * 授权开始
     * @param platformType 目标平台
     */
    @Override
    public void onStart(PlatformType platformType) {
    }

    /** 
     * 授权成功
     * @param platformType  目标平台
     * @param oAuthUserInfo 用户信息
     */
    @Override
    public void onComplete(PlatformType platformType, UMengOAuthUserInfo oAuthUserInfo) {
    }

    /** 
     * 授权失败
     * @param platformType 目标平台
     * @param errorMsg     错误信息
     */
    @Override
    public void onError(PlatformType platformType, String errorMsg) {
    }

    /** 
     * 授权取消
     * @param platformType 目标平台
     */
    @Override
    public void onCancel(PlatformType platformType) {
    }
    
    /** 
     * 设备不支持该平台
     * @param platformType 目标平台
     * @param msg          提示信息
     */@Override
    public void onNotSupported(PlatformType platformType, String msg) {
    }
    
    /** 
     * 权限被拒绝
     * @param platformType 目标平台
     * @param permissions  被拒绝的权限列表
     */
    @Override
    public void onPermissionDenied(PlatformType platformType, String... permissions) {
    }
    
}
```
+++
### 4.3 用户信息
授权成功返回用户基本信息
```java
/**
 * 第三方登录授权获取的用户信息
 */
public class UMengOAuthUserInfo {
  
    private String uid;                                     // 用户id
  
    private String openid;                                  // openid（微信）
  
    private String unionid;                                 //unionid（微信）
                                                            //微信返回的openID和unionID都可以实现用户标识的需求
                                                            //二者的区别在于，unionID可以实现同一个开发者账号下的应用之间账号打通的需求
  
    private String name;                                    // 用户名
  
    private String accessToken;                             // 访问用token
  
    private String RefreshToken;                            // 刷新用token
  
    private String expiration;                              // token过期时间
  
    private String location;                                // 用户位置
  
    private String iconurl;                                 // 用户头像
  
    private String gender;                                  // 用户性别
  
    private String country;                                 // 国家
  
    private String province;                                // 省份
  
    private String city;                                    // 城市
  
    private String followers_count;                         // 关注数（新浪微博）
  
    private String friends_count;                           // 好友数（新浪微博）
  
    private String is_yellow_year_vip;                      // 是否是黄钻（QQ）
  
    private String yellow_vip_level;                        // 黄钻等级（QQ）
}
```
+++
### 4.4 onActivityResult
授权登录也需要在Activity的onActivityResult传递返回值：
```
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
        
    // 执行回调传递
    UMengTools.onActivityResult(this, requestCode, resultCode, data);
}
```
+++
### 4.5 删除授权
用户退出登录或其他需要清除授权状态：
```
//删除全部平台的授权
UMengOAuth.clearOAuth(Context activity);
  
//删除指定平台的授权
UMengOAuth.clearOAuth(Context activity,PlatformType... platformTypes);
```
+++
### 4.6 销毁
释放相关对象
```
@Override
protected void onDestroy() {
    // 释放友盟对象
    UMengTools.release(this);
        
    super.onDestroy();
}
```
---  
## 五、统计
测试统计前必须先完成以下步骤：
* 在友盟后台中开启App的集成测试功能
  
* 通过友盟助手的设备信息采集接口获取手机的设备信息：
    ```
    UMengTools.getTestDeviceInfo(context);
    ```
* 在友盟后台集成测试功能中添加测试设备，提交上面设备信息接口返回的如下格式数据：
    ```
    {"device_id":"861337030615565","mac":"74:ac:5f:99:79:98"}
    ``` 
+++
友盟统计主要分为以下几种情况：

    一、应用时长
    二、页面跳转
    三、自定义事件
        1.计数事件
        2.计算事件
+++     
### 5.1 权限
友盟统计需要以下系统权限，需要app自行实现权限检测和用户交互：

    1.  <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    
建议在app启动时尽早获取，因为涉及页面统计
+++
### 5.2 应用时长
app的使用时长，onResume和onPause中添加：
```
@Override
protected void onResume() {
    super.onResume();
        
    //这里Activity只统计时长，不统计页面名称。
    //如果不需要统计Activity内fragment或view的跳转，pageName则可以设置为Activity的页面名称
    UMengAnalytics.onPageStart(this, null);
}
  
@Override
protected void onPause() {
    super.onPause();
    
    //这里Activity只统计时长，不统计页面名称
    //如果不需要统计Activity内fragment或view的跳转，pageName则可以设置为Activity的页面名称
    UMengAnalytics.onPageEnd(this, null);
}
```
+++
注意事项：
  
* onPageStart和onPageEnd必须成对调用
  
* 不能线性交叉调用，必须保证前一对调用完成，才能线性开始下一对调用，不然统计数据不准确
+++ 
### 5.3 页面跳转
* 只记录Activity的页面跳转：
```
UMengAnalytics.onPageStart(activity, "activity name");
UMengAnalytics.onPageEnd(activity, "activity name");
```
  
* 记录Activity内的Fragment或View的跳转：
     
        1.对应的Activity只记录停留时长
       
        UMengAnalytics.onPageStart(activity, null);
        UMengAnalytics.onPageEnd(activity, null);
        
        2.Activity内的Fragment或View的跳转时线性成对的调用以下两个方法：
        
        UMengAnalytics.onPageStart("page name");
        UMengAnalytics.onPageEnd("page name");
  ```
+++    
### 5.4 自定义事件
* 计数事件
  
  计数事件是用来记录事件的发生次数，每个事件对应一个唯一的事件ID，需要在友盟后台添加所需要的事件ID：

+++
        1.只发送事件ID：
```
new UMengAnalytics.CountEvent(context)
                            .id(eventId)
                            .record();
```
+++
        2.发送事件ID和事件标签：
```
new UMengAnalytics.CountEvent(context)
                            .id(eventId)
                            .label(label)
                            .record();
```
+++
        3.发送事件ID和事件属性键值对，用来详细描述事件的属性：
```
new UMengAnalytics.CountEvent(context)
                            .id(eventId)
                            .withMap(Map<String,String> extras)
                            .record();
```        
+++
或者
``` 
new UMengAnalytics.CountEvent(context)
                            .id(eventId)
                            .with(key1,value1)
                            .with(key2,value2)
                            .with(key3,value3)
                            .record();

```
+++
* 计算事件
  
  计算事件是用来记录事件的消费时间长度或者有价值的数值统计，例如用户看了某个视频多长秒，等等。每个事件对应有一个32位整形参数：
+++
        1.只发送事件ID和事件值：
```
new UMengAnalytics.CalculateEvent(context)
                            .id(eventId)
                            .duration(value)
                            .record();
```
+++
        2.发送事件ID、事件属性和事件值：
```
new UMengAnalytics.CalculateEvent(context)
                            .id(eventId)
                            .withMap(extras)
                            .duration(value)
                            .record();
```
+++
或者
```
new UMengAnalytics.CalculateEvent(context)
                            .id(eventId)
                            .with(key1,value1)
                            .with(key2,value2)
                            .with(key3,value3)
                            .duration(value)
                            .record();

```
+++ 
### 5.5 杀进程
如果遇到手动杀app进程的情况，需要在kill之前调用如下方法，保证之前的统计数据得到保存：
```
// kill app之前保存统计
UMengAnalytics.onKillProcess(context);
...
System.exit(0);
...
```
+++  
### 5.6 其他注意事项
如果需要按照app自身账号体系来统计数据，则参考以下方式：
+++
* app启动时读取之前的登录状态，如果已经登录，则在友盟助手初始化时即可绑定之前登录的账户：

```
UMengTools.create(applicationContext)
            ...
            .setAnalyticsWithUser("10035")                          // 按app自有账号系统进行统计
            .setAnalyticsWithUser("QQ","40745050")                  // 或者如果之前是第三方登录的，则可以绑定第三方的用户账号
            ...
```
+++
* 若app启动时未登录，则用户登录后，可以调用以下方法进行统计账号的绑定：

```
UMengAnalytics.setAnalyticsWithUser("10035");                       // 按app自有账号系统进行统计
UMengAnalytics.setAnalyticsWithUser("QQ","40745050");               // 或者如果是第三方登录的，则可以绑定第三方的用户账号
```
+++
* 若账号退出时需要解绑统计账号，则可以如下调用进行统计账号的解绑：

```
UMengAnalytics.setAnalyticsWithoutUser();                           // 解绑统计账号
```
---
完

Thanks
         
         
            
           
  


