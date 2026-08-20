# FirebaseUI Auth for iOS (SwiftUI)

FirebaseUI provides drop-in SwiftUI views for the whole sign-in flow, so you
don't hand-build sign-in screens on top of the `FirebaseAuth` SDK. Use it when
the app needs a working auth UI quickly; use [client SDK setup](ios_setup.md)
directly when the UI must be fully custom.

Requires iOS 17+ and Swift 6.0+.

Canonical docs:
https://github.com/firebase/FirebaseUI-iOS/blob/main/FirebaseSwiftUI/README.md

## 1. Add the Package

In Xcode, **File > Add Package Dependencies...** and enter the package URL
`https://github.com/firebase/FirebaseUI-iOS`, then pick the products you need:

- `FirebaseAuthSwiftUI` (required, includes the email provider)
- `FirebaseGoogleSwiftUI`, `FirebaseAppleSwiftUI`, `FirebaseFacebookSwiftUI`,
  `FirebaseTwitterSwiftUI`, `FirebasePhoneAuthSwiftUI`
- `FirebaseOAuthSwiftUI` for generic OAuth providers (GitHub, Microsoft, Yahoo)

## 2. Configure Firebase

`FirebaseApp.configure()` must run before any FirebaseUI view is created.

```swift
import FirebaseAuthSwiftUI
import FirebaseCore
import SwiftUI

class AppDelegate: NSObject, UIApplicationDelegate {
  func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    FirebaseApp.configure()
    return true
  }
}

@main
struct YourApp: App {
  @UIApplicationDelegateAdaptor(AppDelegate.self) var appDelegate

  var body: some Scene {
    WindowGroup {
      ContentView()
    }
  }
}
```

## 3. Show the Auth UI

Build an `AuthService`, register providers on it, and wrap your authenticated
content in `AuthPickerView`. `AuthPickerView` renders the sign-in flow while the
user is unauthenticated and your content once they are signed in.

```swift
import FirebaseAuthSwiftUI
import FirebaseGoogleSwiftUI
import SwiftUI

struct ContentView: View {
  let authService: AuthService

  init() {
    let configuration = AuthConfiguration()

    authService = AuthService(configuration: configuration)
      .withEmailSignIn()
      .withGoogleSignIn()
  }

  var body: some View {
    AuthPickerView {
      Text("Welcome to your app!")
    }
    .environment(authService)
  }
}
```

Read `authService.authenticationState` to branch on auth state, and set
`authService.isPresented = true` to present the flow manually.

`AuthConfiguration` also accepts `shouldAutoUpgradeAnonymousUsers`, `tosUrl`,
`privacyPolicyUrl`, `emailLinkSignInActionCodeSettings`, and `mfaEnabled`.

For custom buttons, custom navigation, reauthentication, and the full
`AuthService` API, see the README linked above.
