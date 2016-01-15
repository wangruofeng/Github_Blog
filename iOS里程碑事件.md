# iOS里程碑事件

> 记录iOS各种重要里程碑事件


## 私有成员变量的实现
1.0时代，在.h文件采用`@private`关键词

	@interface ViewController : UIViewController {
	    @private
	    NSInteger _value;
	}
	
2.0时代 通过在.m文件通过`匿名Category`

	@interface ViewController ()
	
	@property (nonatomic) NSInteger value;
	
	@end
	
3.0时代 2013 年的 WWDC 允许在 .m 的 `@implementation `中

	@implementation ViewController {
	    NSInteger _value;
	}


## ARC推出
2011

## 自动生成getter和setter方法的`@synthesize`
2012

## iOS 2.1 to iOS 2.2 API Differences

Added frameworks:

* AVFoundation

## iOS 2.2 to iOS 3.0 API Differences

Added frameworks:

* CoreData
* ExternalAccessory
* GameKit
* MapKit
* MessageUI
* MobileCoreServices
* StoreKit

## iOS 3.2 to iOS 4.0 API Differences

Added frameworks:

* Accelerate
* AssetsLibrary
* CoreMedia
* CoreMotion
* CoreTelephony
* CoreVideo
* EventKit
* EventKitUI
* iAd
* ImageIO
* QuickLook

## iOS 4.3 to iOS 5.0 API Differences
Added frameworks

* Accounts
* CoreBluetooth
* CoreImage
* GLKit
* GSS
* NewsstandKit
* Twitter

## 5.1 to iOS 6.0 API Differences
Added frameworks

* AdSupport
* MediaToolbox
* PassKit
* Social

## 6.1 to iOS 7.0 API Differences
Added frameworks

* GameController
* JavaScriptCore
* MediaAccessibility
* MultipeerConnectivity
* SafariServices
* SpriteKit

## iOS 7.1 to iOS 8.0 API Differences
Added frameworks

* Accelerate
* Accounts
* AddressBook
* AddressBookUI
* AudioToolbox
* AudioUnit
* AVFoundation
* AVKit (Added)
* CFNetwork
* CloudKit (Added)
* CoreAudio
* CoreAudioKit (Added)
* CoreAuthentication (Added)
* CoreBluetooth
* CoreData
* CoreFoundation
* CoreImage
* CoreLocation
* CoreMedia
* CoreMotion
* CoreText
* CoreVideo
* EventKit
* EventKitUI
* ExternalAccessory
* Foundation
* GameController
* GameKit
* GLKit
* GSS
* HealthKit (Added)
* HomeKit (Added)
* iAd
* ImageIO
* IOKit
* JavaScriptCore
* LocalAuthentication (Added)
* MapKit
* MediaAccessibility
* MediaPlayer
* MessageUI
* Metal (Added)
* MobileCoreServices
* MultipeerConnectivity
* NetworkExtension (Added)
* NewsstandKit
* NotificationCenter (Added)
* OpenGLES
* PassKit
* Photos (Added)
* PhotosUI (Added)
* PushKit (Added)
* QuartzCore
* QuickLook
* SceneKit (Added)
* Security
* Social
* SpriteKit
* StoreKit
* UIKit
* VideoToolbox
* WebKit (Added)

## iOS 8.3 to iOS 9.0 API Differences
Added frameworks


Objective-C

* /usr/include
* Accelerate
* Accounts
* AddressBook
* AddressBookUI
* AssetsLibrary
* AudioToolbox
* AudioUnit
* AVFoundation
* AVKit
* CFNetwork
* CloudKit
* Contacts (Added)
* ContactsUI (Added)
* CoreAudio
* CoreAudioKit
* CoreBluetooth
* CoreData
* CoreFoundation
* CoreGraphics
* CoreImage
* CoreLocation
* CoreMedia
* CoreMIDI
* CoreMotion
* CoreSpotlight (Added)
* CoreTelephony
* CoreText
* CoreVideo
* EventKit
* EventKitUI
* ExternalAccessory
* Foundation
* GameController
* GameKit
* GameplayKit (Added)
* GLKit
* GSS
* HealthKit
* HomeKit
* iAd
* ImageIO
* JavaScriptCore
* LocalAuthentication
* MapKit
* MediaPlayer
* MediaToolbox
* MessageUI
* Metal
* MetalKit (Added)
* MetalPerformanceShaders (Added)
* MobileCoreServices
* ModelIO (Added)
* MultipeerConnectivity
* NetworkExtension
* NewsstandKit
* OpenAL
* PassKit
* Photos
* PushKit
* QuartzCore
* QuickLook
* ReplayKit (Added)
* SafariServices
* SceneKit
* Security
* SpriteKit
* StoreKit
* SystemConfiguration
* UIKit
* VideoToolbox
* WatchConnectivity (Added)
* WatchKit
* WebKit

备注：欢迎转载，但请一定注明出处！ <https://github.com/wangruofeng/Github_Blog>