# Thanx iOS SDK

The Thanx iOS SDK provides a simple way to embed the Thanx customer engagement
experience in iOS applications.

## Delivery

The SDK will be delivered as a native Swift library distributed via the package
manager Cocoapods.

## Authentication

### Merchant

Thanx will provide the SDK `Client ID` and a `Client Secret` via a secure
channel.

Authentication of the SDK should be done with that Client ID and Client Secret.
The developer, should initialize the SDK when the app launches with those
values.

### User

Once the user logs into the app, the user's details (email, name, developer
user identifier, Thanx user identifier, etc) should be set in the SDK.

If no Thanx user identifier is provided during authentication, the SDK then
tries to match to a Thanx user via email only and fires a callback with success
or failure:
* If match success, the SDK will return the Thanx user identifier and the
  developer should save that in order to authenticate the user in subsequent
  sessions.
* If match failure, the SDK will present the user with the Thanx sign up flow in
  order to register/login into the Thanx platform. After the user completes the
  Thanx registration/authentication, the SDK will fire the callbacks again.

## Push Notifications

Thanx will need the push services certificate in order to send Loyalty related
push notifications to the user. The Thanx team will work with Westfield to
provide a mechanism for securely exchanging this information.

In the app, after the developer asks for push notifications permissions and the
user accepts, the developer should send the push registration token to the SDK
so the user gets the loyalty related push notifications that the Thanx platform
offers.

## User Experience

### Web flows

Thanx SDK user experience will be web based. When the user launches the rewards,
they'll be presented with a web view that will load the Thanx platform. There,
the user will be able to see and redeem rewards and link cards in order to
accrue progress when shopping at the stores.

## Integration Testing

The SDK will provide an option to run in test mode. In that mode, the web view
and the API will point to a dev environment where the developer will be able to
test the integration without affecting production data (more details when SDK
is delivered).

## Installation

#### Via Cocoapods

The ThanxSDK pod lives in [Thanx's Spec repo](https://github.com/thanx/Specs),
for this reason, the first thing you need to add to your **Podfile** is the repo
url as a new source:

```ruby
# Podfile
source 'https://github.com/thanx/Specs.git'
```

Then you can include the *ThanxSDK* pod as one of your dependencies:

```ruby
# Podfile
source 'https://github.com/thanx/Specs.git'

target 'Your App' do
  use_frameworks!

  # Thanx SDK pod
  pod 'ThanxSDK'

  # ... Other dependencies ...
end
```

The SDK uses the camera and photo library in order to upload receipts so make
sure to add the following in your plist:

```
<key>NSCameraUsageDescription</key>
<string>This app requests access to your camera and photo library to allow you to easily add credit cards (by reading them with your camera) as well as upload photos of receipts.</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>This app requests access to your camera and photo library to allow you to easily add credit cards (by reading them with your camera) as well as upload photos of receipts.</string>
```

## Initialization

You should initialize the SDK in the **AppDelegate** file of your project:

1. Import the SDK at the top of the file: **import ThanxSDK**
2. Call **Thanx.initialize** on the
  **application:didFinishLaunchingWithOptions** method

**Swift**:

```swift
// AppDelegate.swift
import ThanxSDK

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

  var window: UIWindow?

  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    Thanx.initialize(clientId: "YOUR_CLIENT_ID", clientSecret: "YOUR_CLIENT_SECRET", debug: true)
    return true
  }
}
```

The Thanx SDK is now initialized and ready for use.

<aside class="notice">
  Make sure to set <strong>debug: false</strong> for production builds.
</aside>

## Launching the Thanx experience

The Thanx SDK provides the **WebViewController** with the full Thanx mobile
experience. The UIViewController can be presented in any way. Example:

**Swift**:

```swift
// UIViewController.swift
# Present it as a full screen modal
let thanxWebViewController = Thanx.WebViewController(showToolbar: true)
present(thanxWebViewController, animated: true)

# Present it as a part of a UINavigationController
let thanxWebViewController = Thanx.WebViewController(showToolbar: false)
navigationController?.pushViewcontroller(thanxWebViewController, animated: true)
```

## Push Notifications

### Device Token Registration

Thanx will send some push notifications to the user to remind him/her about
their progress towards their reward, to leave feedback, and when they achieve a
new reward. To make it work, the SDK needs to be notified about the device
token whenever the user registers a new device for push notifications:

**Swift**:

```swift
// AppDelegate.swift
import ThanxSDK

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
  func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    Thanx.registerForNotifications(token: deviceToken)
  }
}
```

### Push Notification Landing Screens

Thanx also delivers some contextual landing screens for most of the push
notifications. These screens are meant to give a bit more context to the user
about what just happened with their rewards and also contain very engaging call
to actions with very high yield (for example, to leave feedback about the most
recent visit). In order to enable these landing screens, the SDK needs to be
notified every time a new push notification arrives.

**Swift**:

```swift
// AppDelegate.swift
import ThanxSDK

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    // Make sure that the SDK is initialized before launching the WebViewController
    Thanx.initialize(clientId: "YOUR_CLIENT_ID", clientSecret: "YOUR_CLIENT_SECRET", debug: true) { error in
      guard error == nil else { return }
      if let notificationPayload = launchOptions?[UIApplicationLaunchOptionsKey.remoteNotification] as? [AnyHashable: Any] {
        // Initialize the WebViewController with the notification payload from launchOptions?[UIApplicationLaunchOptionsKey.remoteNotification]
        let webViewController = Thanx.WebViewController(showToolbar: true, notificationPayload: notificationPayload)
        // Present the WebViewController however you want
        window?.rootViewController?.present(webViewController, animated: true)
      }
    }
    return true
  }
}
```

<aside class="notice">
  <strong>
    Make sure that the SDK is initialized before presenting the WebViewController
  </strong>
</aside>

## Debug Mode

In order to run the SDK in debug mode, you'll need to initialize it with the
debug flag as true:

```swift
// AppDelegate.swift
Thanx.initialize(clientId: "YOUR_CLIENT_ID", clientSecret: "YOUR_CLIENT_SECRET", debug: true)
```

Running the SDK in debug will provide you with extra console output and most
importantly, it will point to the **Thanx Staging environment**.

**You should always run in Debug mode unless is a production build.**

## Class Reference

```swift
//! Project version number for ThanxSDK.
public var ThanxSDKVersionNumber: Double
public struct Thanx { }

extension Thanx {
  /// ViewController with the Thanx experience as a WebView
  public class WebViewController : UIViewController {
    /**
    Initializes the WebViewController to display the Thanx experience within a WKWebView

    - Parameters:
    - showToolbar: Whether or not to display the toolbar (You might want to display it when ViewController is
    presented as a full screen modal)
    */
    public convenience init(showToolbar: Bool)
  }
}

extension Thanx {
  public typealias LoginCallback = (ThanxSDK.Thanx.User?, Error?) -> ()

  /// The authenticated user
  public static var user: ThanxSDK.Thanx.User?

  /// Whether or not the user is authenticated.
  public static var authenticated: Bool

  /**
  Logs the user out

  - Parameters:
  - completion: Callback fired after log out has completed
  */
  public static func logout(completion: (() -> ())? = default)
}

extension Thanx {
  /// Whether or not the SDK has been initialized
  public static var initialized: Bool { get }

  /**
  Initializes the SDK with the client ID and client secret provided.

  - Parameters:
  - clientId: Your SDK client ID
  - clientSecret: Your SDK client secret
  - debug: Whether or not to run the SDK in debug mode
  - completion: Callback fired when the initialization has been completed
  */
  public static func initialize(clientId: String, clientSecret: String, debug: Bool = default, completion: ((Error?) -> ())? = default)
}

extension Thanx {
  public class User {
    public let firstName: String!

    public let lastName: String!

    public let email: String!
  }
}
```
