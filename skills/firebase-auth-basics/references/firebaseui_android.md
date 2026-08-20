# FirebaseUI Auth for Android (Compose)

FirebaseUI Auth is a Compose-based library of drop-in authentication screens
built on Firebase Auth and Material Design 3. Use it when the app needs a
working sign-in flow quickly; use [client SDK setup](client_sdk_android.md)
directly when the UI must be fully custom.

Requires Android SDK 21+, Kotlin 1.9+, Compose compiler 1.5+, and Firebase Auth
22.0.0+.

Canonical docs:
https://github.com/firebase/FirebaseUI-Android/blob/master/auth/README.md

## 1. Add Dependencies

In the module `build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.firebaseui:firebase-ui-auth:10.0.0-beta03")

    implementation(platform("com.google.firebase:firebase-bom:32.7.0"))
    implementation("com.google.firebase:firebase-auth")

    implementation(platform("androidx.compose:compose-bom:2024.01.00"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.material3:material3")

    // Only if you use AuthProvider.Facebook()
    implementation("com.facebook.android:facebook-login:16.3.0")
}
```

Check the README for the current FirebaseUI version before pinning it.

## 2. Configure Providers

Google Sign-In needs no Android-specific code — the `google-services` Gradle
plugin supplies the configuration, so just enable Google in the Firebase
Console. Twitter, GitHub, Microsoft, Yahoo, and Apple are also Console-only.

Facebook Login additionally needs `facebook_application_id`,
`facebook_login_protocol_scheme`, and `facebook_client_token` in
`res/values/strings.xml`.

## 3. Show the Auth UI

Declare providers with `authUIConfiguration { }` and render
`FirebaseAuthScreen`. It handles sign-in, sign-up, password reset, display name
collection, Credential Manager, and error display.

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            MyAppTheme {
                val configuration = authUIConfiguration {
                    providers {
                        provider(AuthProvider.Email())
                        provider(AuthProvider.Google())
                    }
                }

                FirebaseAuthScreen(
                    configuration = configuration,
                    onSignInSuccess = { result -> /* navigate to your app */ },
                    onSignInFailure = { exception -> /* show an error */ },
                    onSignInCancelled = { finish() },
                )
            }
        }
    }
}
```

## 4. Observe Auth State

`FirebaseAuthUI.getInstance()` is the entry point for auth state and account
operations. Prefer the flow so the UI reacts to sign-out and token changes:

```kotlin
@Composable
fun AuthGate() {
    val authUI = remember { FirebaseAuthUI.getInstance() }
    val authState by authUI.authStateFlow().collectAsState(initial = AuthState.Idle)

    if (authState is AuthState.Success) {
        MainAppScreen()
    } else {
        FirebaseAuthScreen(/* ... */)
    }
}
```

`authUI.isSignedIn()` is available for a one-off check outside composition.

For theming, multi-factor auth, anonymous user upgrade, custom slot-based UI,
and the low-level `AuthFlowController` API, see the README linked above.
