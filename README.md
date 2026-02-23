# Google Sign-In Unity Plugin - Personal Fork

This is a personal fork of the Google Sign-In Unity Plugin, upgraded to use the latest Android and iOS authentication libraries.

**Personal Fork:** [weiping-playnext/google-signin-unity](https://github.com/weiping-playnext/google-signin-unity)  
**Upstream Fork:** [Thaina/google-signin-unity](https://github.com/Thaina/google-signin-unity)  
**Original Repository:** [googlesamples/google-signin-unity](https://github.com/googlesamples/google-signin-unity)

## Migration to New APIs

This fork migrates from deprecated Google Sign-In libraries to the new authentication systems:

- **Android:** Migrated to `CredentialManager` and `AuthorizationClient` ([Migration Guide](https://developer.android.com/identity/sign-in/legacy-gsi-migration))
- **iOS:** Updated to latest GoogleSignIn SDK ([Quick Migration Guide](https://developers.google.com/identity/sign-in/ios/quick-migration-guide))

### Acknowledgments

Thanks to the iOS fix from [pillsgood/google-signin-unity](https://github.com/pillsgood/google-signin-unity) and [@DulgiKim](https://github.com/googlesamples/google-signin-unity/pull/205#issuecomment-1724733615) for their contributions.

## Key Changes in This Fork

### Android Migration
Android has been migrated to use `CredentialManager` and `AuthorizationClient` since [GoogleSignInAccount was deprecated](https://developers.google.com/android/reference/com/google/android/gms/auth/api/signin/GoogleSignInAccount).

**Important Notes:**
- `GoogleIdTokenCredential` no longer provides numeric unique IDs; it uses email as userId instead
- This fork extracts the JWT `sub` value from idToken (consistent with userId from GoogleSignIn on other platforms)
- The new system does not support email hints
- **WebClientId is now required** in addition to Android Client ID and must be provided at configuration initialization

### iOS Migration
The iOS implementation has been updated to support the latest GoogleSignIn SDK (v7.0.0+). An editor tool `PListProcessor` has been added to automatically configure `GIDClientID` and `GIDServerClientID` in Info.plist.

## Configuration Example

Here's how to configure Google Sign-In in your Unity project:

```C#
GoogleSignIn.Configuration = new GoogleSignInConfiguration() {
    RequestEmail = true,
    RequestProfile = true,
    RequestIdToken = true,
    RequestAuthCode = true,
    // IMPORTANT: Must be web client ID, not Android client ID
    WebClientId = "XXXXXXXXX-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.apps.googleusercontent.com",
#if UNITY_EDITOR || UNITY_STANDALONE
    ClientSecret = "XXXXXX-xxxXXXxxxXXXxxx-xxxxXXXXX" // Optional for Windows/macOS and testing in editor
#endif
};
```

**Tested with:** Unity 2021.3.21 and Unity 6000.0.5

## Installation

Add the UPM dependency to your `manifest.json` in the `Packages` folder:

### Stable Branch (Recommended)

```json
{
  "dependencies": {
    "com.google.external-dependency-manager": "https://github.com/googlesamples/unity-jar-resolver.git?path=upm",
    "com.google.signin": "https://github.com/weiping-playnext/google-signin-unity.git#newmigration",
    ...
  }
}
```

### Development Branch

For the latest experimental features and fixes:

```json
{
  "dependencies": {
    "com.google.external-dependency-manager": "https://github.com/googlesamples/unity-jar-resolver.git?path=upm",
    "com.google.signin": "https://github.com/weiping-playnext/google-signin-unity.git#personal_working",
    ...
  }
}
```

## iOS Configuration

[The new version of iOS SDK recommends](https://developers.google.com/identity/sign-in/ios/quick-migration-guide#google_sign-in_sdk_v700) setting `GIDClientID` and `GIDServerClientID` in Info.plist.

This fork includes an editor tool `PListProcessor` that automatically extracts `CLIENT_ID` and `WEB_CLIENT_ID` properties from plist files in your project that match your Unity bundle identifier.

### Setting up iOS Credentials

1. Download the plist file from [Google Cloud Console](https://console.cloud.google.com) credential page
2. Select your iOS credential and click the ⬇ download button
3. Place the downloaded plist file in your Unity project
4. The `PListProcessor` will automatically configure it during build

**Required plist format:**

```xml
<!-- This is the default format downloaded from Google Cloud Console -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>CLIENT_ID</key> 
	<string>{YourCloudProjectID}-yyyyyYYYYyyyyYYYYYYYYYYYYYYyyyyyy.apps.googleusercontent.com</string>
	<key>REVERSED_CLIENT_ID</key>
	<string>com.googleusercontent.apps.{YourCloudProjectID}-yyyyyYYYYyyyyYYYYYYYYYYYYYYyyyyyy</string>
	<key>PLIST_VERSION</key>
	<string>1</string>
	<key>BUNDLE_ID</key>
	<string>com.{YourCompany}.{YourProductName}</string>
	<!-- Optional: Add these lines manually if you need ServerAuthCode -->
	<key>WEB_CLIENT_ID</key>
	<string>{YourCloudProjectID}-zzzZZZZZZZZZZZZZZzzzzzzzzzzZZZzzz.apps.googleusercontent.com</string>
</dict>
</plist>
```

---

## Original Documentation

*The documentation below is from the original Google Sign-In Unity Plugin. Some information may be outdated but is kept for reference.*

# Google Sign-In Unity Plugin
_Copyright (c) 2017 Google Inc. All rights reserved._


## Overview

Google Sign-In API plugin for Unity game engine.  Works with Android and iOS.
This plugin exposes the Google Sign-In API within Unity.  This is specifically
intended to be used by Unity projects that require OAuth ID tokens or server
auth codes.

It is cross-platform, supporting both Android and iOS.

See [Google Sign-In for Android](https://developers.google.com/identity/sign-in/android/start)
for more information.

## Configuring the application  on the API Console

To authenticate you need to create credentials on the API console for your
application. The steps to do this are available on
[Google Sign-In for Android](https://developers.google.com/identity/sign-in/android/start)
or as part of Firebase configuration.
In order to access ID tokens or server auth codes, you also need to configure
a web client ID.

## How to build the sample


### Get a Google Sign-In configuration file
This file contains the client-side information needed to use Google Sign-in.
The details on how to do this are documented on the [Developer website](https://developers.google.com/identity/sign-in/android/start-integrating#get-config).

Once you have the configuration file, open it in a text editor.  In the middle
of the file you should see the __oauth_client__ section:
```
      "oauth_client": [
        {
          "client_id": "411000067631-hmh4e210xxxxxxxxxx373t3icpju8ooi.apps.googleusercontent.com",
          "client_type": 3
        },
        {
          "client_id": "411000067631-udra361txxxxxxxxxx561o9u9hc0java.apps.googleusercontent.com",
          "client_type": 1,
          "android_info": {
            "package_name": "com.your.package.name.",
            "certificate_hash": "7ada045cccccccccc677a38c91474628d6c55d03"
          }
        }
      ]
```

There are 3 values you need for configuring your Unity project:
1. The __Web client ID__.  This is needed for generating a server auth code for
your backend server, or for generating an ID token.  This is the `client_id`
value for the oauth client with client_type == 3.
2. The __package_name__.  The client entry with client_type == 1 is the
Android client.  The package_name must be entered in the Unity player settings.
3.  The keystore used to sign your application. This is configured in the publishing settings of the Android Player properties in
the Unity editor.  This must be the same keystore used to generate
the SHA1 fingerprint when creating the application on the console.  __NOTE:__
The configutation file does not reference the keystore, you need to keep track of
this yourself.


### Create a new project and import the plugin
Create a new Unity project and import the `GoogleSignIn-1.0.0.unitypackage` (or the latest version).
This contains native code, C# Unity code needed to call the Google Sign-In API for both Android and iOS.

### Import the sample scene
Import the `GoogleSignIn-sample.unitypackage` which contains the sample scene and
scripts.  This package is not needed if you are integrating Google Sign-in into
your own application.

### Configure the web client id
1. Open the sample scene in `Assets/SignInSample/MainScene`.
2. Select the Canvas object in the hierarchy and enter the web client id
in the __SignInSampleScript__ component.

## Building for Android
1. Under Build Settings, select Android as the target platform.
2. Set the package name in the player settings to the package_name you found in
the configuration file.
3. Select the keystore file, the key alias, and passwords.
4. Resolve the Google Play Services SDK dependencies by selecting from the menu:
    __Assets/Play Services Resolver/Android Resolver/Resolve__.  This will add
    the required .aar files to your project in `Assets/Plugins/Android`.

## Building for iOS
For iOS, follow the instructions for creating a GoogleService-Info.plist file on
https://developers.google.com/identity/sign-in/ios/start-integrating.

In Unity, after switching to the iOS player, make sure to run the Play Services
Resolver.  This will add the required frameworks and libraries to the XCode
project via CocoPods.

After generating the XCode project from Unity, download the GoogleService-Info.plist file
from the Google Developers website and add it to your XCode project.

## Using the Games Profile to sign in on Android
To use the Play Games Services Gamer profile when signing in, you need to edit the
dependencies in `Assets/GoogleSignIn/Editor/GoogleSignInDependencies.xml`.

Uncomment the play-services-games dependency and re-run the resolution.


## Using this plugin with Firebase Auth
Follow the instructions to use Firebase Auth with Credentials on the [Firebase developer website]( https://firebase.google.com/docs/unity/setup).

Make sure to copy the google-services.json and/or GoogleService-Info.plist to your Unity project.

Then to use Google SignIn with Firebase Auth, you need to request an ID token when authenticating.
The steps are:
1. Configure Google SignIn to request an id token and set the web client id as described above.
2. Call __SignIn()__ (or __SignInSilently()__).
3. When handling the response, use the ID token to create a Firebase Credential.
4. Call Firebase Auth method  __SignInWithCredential()__.

```
    GoogleSignIn.Configuration = new GoogleSignInConfiguration {
      RequestIdToken = true,
      // Copy this value from the google-service.json file.
      // oauth_client with type == 3
      WebClientId = "1072123000000-iacvb7489h55760s3o2nf1xxxxxxxx.apps.googleusercontent.com"
    };

    Task<GoogleSignInUser> signIn = GoogleSignIn.DefaultInstance.SignIn ();

    TaskCompletionSource<FirebaseUser> signInCompleted = new TaskCompletionSource<FirebaseUser> ();
    signIn.ContinueWith (task => {
      if (task.IsCanceled) {
        signInCompleted.SetCanceled ();
      } else if (task.IsFaulted) {
        signInCompleted.SetException (task.Exception);
      } else {

        Credential credential = Firebase.Auth.GoogleAuthProvider.GetCredential (((Task<GoogleSignInUser>)task).Result.IdToken, null);
        auth.SignInWithCredentialAsync (credential).ContinueWith (authTask => {
          if (authTask.IsCanceled) {
            signInCompleted.SetCanceled();
          } else if (authTask.IsFaulted) {
            signInCompleted.SetException(authTask.Exception);
          } else {
            signInCompleted.SetResult(((Task<FirebaseUser>)authTask).Result);
          }
        });
      }
    });
```


## Building the Plugin
To build the plugin run `./gradlew -PlintAbortOnError build_all`. This builds the support aar
library with lint warnings as errors and packages the plugin into a .unitypackage file.  It
also packages the sample scene and script in a separate package.

There's also a shortcut for linux/mac: `./build_all`.


## Questions? Problems?

For issues related to this personal fork, please open an issue on the [personal fork repository](https://github.com/weiping-playnext/google-signin-unity).

For upstream fork questions, refer to [Thaina/google-signin-unity](https://github.com/Thaina/google-signin-unity).  
For general Google Sign-In Unity questions, refer to the [original repository](https://github.com/googlesamples/google-signin-unity).

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests to improve this fork.

## License

This project maintains the original license from the Google Sign-In Unity Plugin.  
_Original Copyright (c) 2017 Google Inc. All rights reserved._
