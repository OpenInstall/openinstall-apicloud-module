# openinstall-apicloud-module

<div class="outline">

[config](#config)

[init](#init)

[getWakeup](#getWakeup)

[getInstall](#getInstall)

[reportRegister](#reportRegister)

[reportEffectPoint](#reportEffectPoint)

</div>

# **概述**

openinstallsdk 封装了openinstall平台的SDK，集成了渠道统计,携带参数安装,快速下载和一键拉起功能；可用于实现移动广告效果统计,免填邀请码,安装后自动加好友,一键加入游戏房间,用户分享统计,微信中快速下载和一键拉起等，根据需求可实现更多场景。

# **初始化配置**

**使用之前须从openinstall平台申请开发者账号并创建应用，获取AppKey，使用此模块之前建议先配置config.xml 文件，配置完毕，需通过云端编译生效，配置方法如下：**

- 参数：urlScheme、appKey
- 配置示例:

```xml
<permission name="internet" />
<preference name="urlScheme" value="openinstall官方自动分配的scheme" />
//android下
<meta-data name="com.openinstall.APP_KEY" value="openinstall官方自动分配的appKey" />
//iOS下
<feature name="openinstall">
    <param name="com.openinstall.APP_KEY" value="openinstall官方自动分配的appKey" />
</feature>

```
- 字段描述:
    **internet**：添加网络权限；
    **urlScheme**：使用一键拉起功能必须配置，urlScheme 的 value 值详细获取位置：openinstall应用控制台-> Android集成-> Android应用配置，iOS同理；
    **com.openinstall.APP_KEY**：（必须配置）从openinstall平台获取的 AppKey。


# **universal links相关配置（针对iOS）**

- 开启Associated Domains服务

对于iOS，为确保能正常跳转，AppID必须开启Associated Domains功能，请到[苹果开发者网站](https://developer.apple.com)，选择Certificate, Identifiers & Profiles，选择相应的AppID，开启Associated Domains。注意：当AppID重新编辑过之后，需要更新相应的mobileprovision证书。(图文配置步骤请看[iOS集成指南](https://www.openinstall.io/doc/ios_sdk.html))。更新mobileprovision证书步骤请查看[云编译mobileprovision证书制作](https://docs.apicloud.com/Dev-Guide/iOS-License-Application-Guidance) 中的 "云编译mobileprovision发布证书制作"或"云编译mobileprovision测试证书制作"。

- 配置universal links关联域名  

`关联域名(Associated Domains)`的值请在openinstall控制台获取（openinstall应用控制台->iOS集成->iOS应用配置）  

该文件是给iOS平台配置的文件，在widget\res下创建文件名为UZApp.entitlements的文件，UZApp.entitlements内容如下：

```xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.developer.associated-domains</key><!--固定key值-->
    <array>
     <!--这里换成你在openinstall后台的关联域名(Associated Domains)-->
        <string>applinks:xxxxxx.openinstall.io</string>
    </array>
</dict>
</plist>

```

**openinstall完全兼容微信openSDK1.8.6以上版本的通用链接跳转功能，注意在使用 微信模块wxPlus或wxpayPlus 时，微信要求配置的config.xml，请传入正确格式的universal link链接。链接格式参考[iOS常见问题](https://www.openinstall.io/doc/ios_sdk_faq.html)**

# **API调用**


## 初始化配置

<div id="config"></div>

# **config**
Android 配置接口，设置广告平台相关参数

config(options)

## options

adEnabled: 
- 类型：布尔类型；
- 描述：表示 openinstall SDK 是否需要获取广告追踪相关参数，默认为 false  

oaid: 
- 类型：字符串；
- 描述：通过移动安全联盟获取到的 oaid，默认为 null  

gaid: 
- 类型：字符串；
- 描述：通过 google api 获取到的 advertisingId，默认为 null  

## 示例代码

```js
var openinstall = api.require('openinstall');
openinstall.config({
  adEnabled: true,
  oaid: null,
  gaid: null,
});

```

## 补充说明
此接口是 Android 平台针对广告平台接入而新增的配置接口，需要在调用 init 之前调用。参考 [广告平台对接Android集成指引](https://www.openinstall.io/doc/ad_android.html)

## 可用性
Android系统,iOS系统

可提供的1.3.2及更高版本

## 初始化
<div id="init"></div>

# **init**

## 示例代码
``` js
var openinstall = api.require('openinstall');
openinstall.init();
```

## 可用性

Android系统,iOS系统

可提供的 1.3.0 及更高版本

## 补充说明
如启用了广告平台对接，请按以下操作初始化

对于Android平台，需要在 `config.xml` 添加权限申明 `<permission name="readPhoneState"/>`

对于iOS平台，获取idfa使用的是apicloud官方模块iAd，需要用户主动添加，[iAd模块](http://www.apicloud.com/mod_detail/67745)  


openinstall模块初始化示例代码如下：
``` js
apiready = function() {
		var openinstall = api.require('openinstall');
		if (api.systemType == "ios") {
            //需要用到idfa时，先获取idfa，再初始化
		    var iAd = api.require('iAd');//引入iAd模块
	   	    var iAd = iAd.getIDFA({
	  	        lowerCase: false
		    }, function(ret) {
                	openinstall.init({
                            deviceId:ret.IDFA
                        });
		    });
		} else if (api.systemType == "android") {
		    //申请权限并初始化
	    	    var permissions = [];
	    	    permissions.push('phone-r');
	    	    var result = api.hasPermission({
	       	        list: permissions
	    	    });
	    	    if(result && result[0] && !result[0].granted){
	                //没有权限，去申请
	                api.requestPermission({
                        list: permissions,
                        code: 100
                    }, function(ret, err){
                        //不管是否申请到权限都要调用初始化
                        openinstall.init();
                    });
	    	    }else{
	      	        openinstall.init();
	    	    }
		}

 };
```

## 快速下载  
如果只需要快速下载功能，无需其它功能（携带参数安装、渠道统计、一键拉起），完成初始化即可。

## 一键拉起

<div id="getWakeup"></div>

# **getWakeup**
在拉起APP时，获取由web网页中传递过来的参数

getWakeup({uri:ret},callback(ret, err))

## callback(ret, err)

ret：

- 类型：JSON对象
- 内部字段：

```js
{
    channelCode: '渠道编号',//渠道编号
    data:    '唤醒携带的参数'  //有携带参数，则返回数据，没有则为空
}
```

## 示例代码

```js
var openinstall = api.require('openinstall');
api.addEventListener({
    name: 'appintent'
}, function(ret, err) {
    openinstall.getWakeup({
        "uri": ret
    }, function(ret, err) {
        alert(JSON.stringify(ret));
    });
});

```


## 补充说明
此接口用于获取动态唤醒参数，通过动态参数，在拉起APP时，获取由web网页中传递过来的，如邀请码、游戏房间号等自定义参数，跳转指定页面
监听appintent事件，调用以上代码,获取web端传过来的自定义参数,并回调给getWakeup方法调用;


## 可用性
Android系统,iOS系统

可提供的1.0.0及更高版本

## 携带参数安装（高级版功能）
<div id="getInstall"></div>

# **getInstall**
获取由web网页中传递过来的安装参数  
getInstall({params},callback(ret, err))

## params
timeout：
- 类型：数字类型
- 描述：超时时长，单位秒(s)，默认为10秒


## callback(ret, err)

ret：

- 类型：JSON对象
- 内部字段：

```js
{
    channelCode: '渠道编号',//渠道编号
    data:    '个性化安装携带的参数' 
}
```

## 示例代码

```js
var openinstall = api.require('openinstall');
openinstall.getInstall({
   timeout:10
},function(ret, err){
   alert(JSON.stringify(ret));
});

```

## 补充说明

此接口用于获取动态安装参数（可重复获取），测试时候建议卸载再安装正确获取参数,在APP需要个性化安装参数时（由web网页中传递过来的，如邀请码、游戏房间号等自定义参数），在回调中获取参数，可实现跳转指定页面、统计渠道数据等

## 可用性

Android系统,iOS系统

可提供的1.0.0及更高版本


## 渠道统计（高级版功能）

<div id="reportRegister"></div>

# **reportRegister**

上报注册量  
reportRegister()

## 示例代码

```js
var openinstall = api.require('openinstall');
openinstall.reportRegister();

```
## 补充说明

openinstall 会自动完成安装量、留存率、活跃量、在线时长等渠道统计数据的上报工作,如需统计每个渠道的注册量（对评估渠道质量很重要），可根据自身的业务规则，在确保用户完成app注册的情况下，调用reportRegister()上报注册量。
在openinstall平台即可看到注册量。

## 可用性

Android系统,iOS系统

可提供的1.0.0及更高版本


<div id="reportEffectPoint"></div>

# **reportEffectPoint**

效果点统计

reportEffectPoint({params})

## params
effectId：
- 类型：字符串
- 描述：效果点ID

effectValue：
- 类型：数字类型
- 描述：效果点值，货币以分为单位



## 示例代码

```js
var openinstall = api.require('openinstall');
openinstall.reportEffectPoint({
  effectId:'effect_test',
  effectValue:1
});

```

## 补充说明

openinstall 调用reportEffectPoint({params})统计自定义效果点。  
effectID与effectValue对应的值与openinstall平台的效果点管理的效果点名称与效果点ID必须一一对应。在openinstall平台即可看到渠道管理的渠道效果点。

## 可用性

Android系统,iOS系统

可提供的1.0.0及更高版本
