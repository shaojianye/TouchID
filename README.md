前言：如果图片看不了请移步：[简书](http://www.jianshu.com/p/862c10ee95e3)

## Touch ID简介

---
- `Touch ID`指纹识别作为iPhone 5s上的“杀手级”功能早已为人们所熟知，目前搭载的设备有`iphone SE、iPhone 6、iPhone 6 Plus、iPhone 6s、iPhone 6s Plus、iPad Pro、iPad mini 4、iPad mini 3和iPad air 2`
- `iOS 8.0`开放了指纹验证的API，为APP增添了新的解锁姿势，Touch ID在iPhone 6、iPhone 6 Plus上表现平平，识别效率低下成为众多用户的吐槽点
- 苹果在2015新品发布会上提及全新的`iPhone 6s、iPhone 6s Plus`采用第二代Touch ID，新的Touch ID识别速度更快；实际体验中只要轻轻触碰一下即可，以往要按压半秒钟的指纹识别过程，现在基本是一触即发
- 随着安卓也有越来越多的设备配备了类似Touch ID的指纹识别装置，今后会有越来越多的APP选择使用`指纹识别`的验证方式

## Touch ID原理

---
- Touch ID不存储用户的任何指纹图像，只保存代表指纹的数字字符。iPhone 5s的A7处理器采用了新的高级安全架构，其中有一块名为Secure Enclave的区域用以专门保护密码和指纹数据。只有Secure Enclave可以访问指纹数据，而且它还把这些数据同处理器和系统隔开，因而这些永远不会被存储在苹果的服务器上，也不会被同步到iCloud或其他地方。除了Touch ID之外，它们不会被匹配到其他指纹库中
- 相信很多人都知道，一个Touch ID传感器和iPhone是一对一的关系，如果损坏，只能售后，无法自行更换，其中的原理比较复杂。一种可能的解释是苹果阻止了任何Touch ID和Secure Enclave之间的任何数据嗅探和截取，实现了特定处理器配对特定的Touch ID。
- 如果可以随意更换，那么有人将用户的指纹传感器更换，就可以在用户不知情的情况下窃取到指纹数据。苹果的技术降低了这一风险，这意味着不法之徒想要`调包`传感器的话，需要单独破解每台设备，对于重视安全性的用户来说，这个发现当然是个好消息

![Touch ID原理.png](http://upload-images.jianshu.io/upload_images/1923109-c66bb95d40a20acb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Touch ID常用方法与枚举解释

---
- 首先苹果提供了`canEvaluatePolicy:error:`来判断运行的设备是否支持Touch ID

- 如果要让其失效，可以调用`invalidate`,新特征：iOS 9.0和Mac OS 10.11

- 还提供了`evaluatePolicy:localizedReason:reply:`来验证识别的情况，具体类型如下（以下枚举类型出现的情况均已测试并标注）：

	```
	    // 身份验证没有成功，因为用户未能提供有效的凭据(连续3次验证失败时提示)
		LAErrorAuthenticationFailed = kLAErrorAuthenticationFailed,
				
    	// 身份验证被用户取消（当用户点击取消按钮时提示）
    	LAErrorUserCancel           = kLAErrorUserCancel,
    	    	
    	// 认证被取消了，因为用户点击回退按钮（当用户点击输入密码时提示）
    	LAErrorUserFallback         = kLAErrorUserFallback,
    
    	// 身份验证被系统取消（验证时当前APP被移至后台或者点击了home键导致验证退出时提示）
    	LAErrorSystemCancel         = kLAErrorSystemCancel,
    
    	// Touch ID无法启动，因为没有设置密码（当系统没有设置密码的时候，Touch ID也将不会开启）
    	LAErrorPasscodeNotSet       = kLAErrorPasscodeNotSet,
    
    	// 无法启动身份验证（这种情况没有检测到，应该是出现硬件损坏才会出现）
    	LAErrorTouchIDNotAvailable  = kLAErrorTouchIDNotAvailable,
    
    	// 无法启动身份验证，因为触摸没有注册的手指 （这个暂时没检测到）
    	LAErrorTouchIDNotEnrolled   = kLAErrorTouchIDNotEnrolled,
    
    	// 身份验证是不成功的，因为有太多的失败会要求密码解除锁定，（前提是使用 LAPolicyDeviceOwnerAuthenticationWithBiometrics）iOS9和MAC OS0.11新特征
    	LAErrorTouchIDLockout   NS_ENUM_AVAILABLE(10_11, 9_0) = kLAErrorTouchIDLockout,
    
    	// 认证被取消的应用（如无效而认证进行调用）这个暂时没有检测到，可能是苹果预留的 iOS9和MAC OS0.11新特征
    	LAErrorAppCancel        NS_ENUM_AVAILABLE(10_11, 9_0) = kLAErrorAppCancel,
    
    	// LAContext通过这个电话已经失效（当LAContext失效时会调用）iOS9和MAC OS0.11新特征
    	LAErrorInvalidContext   NS_ENUM_AVAILABLE(10_11, 9_0) = kLAErrorInvalidContext
	
	```


## Touch 使用

---
- 首先，我们需要引入 `LocalAuthentication` 框架

	```
		#import <LocalAuthentication/LocalAuthentication.h>

	```
- 使用很简单，先创建一个`LAContext`对象并配置必要的信息
	
	```
		LAContext *context = [[LAContext alloc] init];
		// 当指纹识别失败一次后，弹框会多出一个选项，而这个属性就是用来设置那个选项的内容
    	context.localizedFallbackTitle = @"使用密码登录";
	
	```
- 配置好LAContext对象后，就需要判断一下设备`是否支持指纹识别功能`

	```
	NSError *error = nil;
	
	if ([context canEvaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics error:&error]) { // 该设备支持指纹识别
	
	}else {
	
	}
	
	```
- 当设备支持指纹识别的时候，实现如下

	```
        [context evaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics localizedReason:@"身份验证需要解锁指纹识别功能" reply:^(BOOL success, NSError * _Nullable error) {
            if (success) {  // 验证成功
                
            }else {
                
                NSLog(@"%@", error.localizedDescription);
                switch (error.code) {
                    case LAErrorSystemCancel:
                        NSLog(@"身份验证被系统取消（验证时当前APP被移至后台或者点击了home键导致验证退出时提示）");
                        break;
                    case LAErrorUserCancel:
                        NSLog(@"身份验证被用户取消（当用户点击取消按钮时提示）");
                        break;
                    case LAErrorAuthenticationFailed:
                        NSLog(@"身份验证没有成功，因为用户未能提供有效的凭据(连续3次验证失败时提示)");
                        break;
                    case LAErrorPasscodeNotSet:
                        NSLog(@"Touch ID无法启动，因为没有设置密码（当系统没有设置密码的时候，Touch ID也将不会开启）");
                        break;
                    case LAErrorTouchIDNotAvailable:
                        NSLog(@"无法启动身份验证");  // 这个没有检测到，应该是出现硬件损坏才会出现
                        break;
                    case LAErrorTouchIDNotEnrolled:
                        NSLog(@"无法启动身份验证，因为触摸标识没有注册的手指");  // 这个暂时没检测到
                        break;
                    case LAErrorUserFallback:
                    {
                        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
                            NSLog(@"用户选择输入密码，切换主线程处理");
                        }];
                        break;
                    }
                    default:
                    {
                        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
                            NSLog(@"其他情况，切换主线程处理");   // 5次失败进入,如果继续验证，则需要输入密码解锁
                        }];
                        break;
                    }
                }
            }
          }
		];
	```
- 如果不支持，实现如下

	```
		NSLog(@"不支持指纹识别");
        	switch (error.code) {
            	case LAErrorTouchIDNotEnrolled:
                	NSLog(@"设备Touch ID不可用");
                	break;
            	case LAErrorPasscodeNotSet:
                	NSLog(@"系统未设置密码");
                	break;
            	default:
                	NSLog(@"TouchID不可用或已损坏");
                	break;
        }
        
        NSLog(@"%@", error.localizedDescription);
	
	```
	效果:![Touch ID效果.gif](http://upload-images.jianshu.io/upload_images/1923109-099b0111a40876fb.gif?imageMogr2/auto-orient/strip)

- [ github-Demo下载请点我](https://github.com/shaojianye/TouchID)—— [如果太慢可以就点我下载](https://git.oschina.net/miaomiaoshen/TouchID.git)
