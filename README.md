# react-native-uiscene-bridge

Launch React Native and Expo apps built with Xcode 27 before they adopt the UIScene life cycle.
Install it and rebuild. No code changes.

If your app closes as soon as it opens after you build it with Xcode 27, and the log shows this
line, this package fixes it:

```
Application failed to launch: UIScene life cycle is required for apps built with this SDK. See "Transitioning to the UIKit scene-based life cycle" in the UIKit documentation for more information on migration.
```

The crash report shows `EXC_BREAKPOINT (SIGTRAP)` in
`___UIApplicationEvaluateRuntimeIssueForNoSceneLifecycleAdoption_block_invoke`.

## Install

React Native:

```sh
npm install react-native-uiscene-bridge
cd ios && pod install
```

Expo, in a development build or an EAS build:

```sh
npx expo install react-native-uiscene-bridge
```

Then build the app again. Autolinking adds the pod. There is no config plugin, and nothing to call
from JavaScript.

## Why Xcode 27 breaks it

Xcode 27 builds with the iOS 27 SDK, and Apple's UIKit documentation says apps built with it "must
adopt the scene-based life cycle or they fail to launch"
([Transitioning to the UIKit scene-based life cycle](https://developer.apple.com/documentation/uikit/transitioning-to-the-uikit-scene-based-life-cycle)).
UIKit checks while the app starts and stops it before `application:didFinishLaunchingWithOptions:`
runs.

React Native's templates create their window in `application:didFinishLaunchingWithOptions:` and
declare no scene. React Native adds scene support in 0.88
([facebook/react-native#57700](https://github.com/facebook/react-native/pull/57700),
[#57717](https://github.com/facebook/react-native/pull/57717)), which is at 0.88.0-rc.2; 0.87 and
earlier have none. Expo SDK 56 and earlier have none either. SDK 57 ships a scene delegate that an
app can switch to, and its templates don't use it yet.

## What it does

- When UIKit sets the app delegate, and the app declares no scene (no `UIApplicationSceneManifest`
  in `Info.plist`, and no `application:configurationForConnectingSceneSession:options:`), the
  bridge adds that method to the delegate's class. It returns a configuration with the bridge's own
  scene delegate, which satisfies UIKit's check.
- The scene delegate puts the window your app delegate already created into the scene and makes it
  key and visible. If there is no window, it creates one and gives it to the app delegate.
- Under scenes, UIKit sends links, user activities, quick actions and foreground and background
  transitions to the scene instead of the app delegate. The bridge passes each one to the app
  delegate method that used to receive it, so `RCTLinkingManager`, Expo's app delegate subscribers
  and your own overrides keep working.
- A link that launches the app now arrives with the scene, not in the launch options. The bridge
  passes it to `application:openURL:options:` as soon as the scene connects, which is where
  expo-linking records the link Expo Router starts from. It also answers React Native's
  `Linking.getInitialURL()` with that link when the launch options have none.
- If the app already declares a scene, the bridge does nothing. Once you adopt scenes yourself,
  with React Native 0.88 or Expo's scene delegate, it steps aside, and you can remove it whenever
  you like.
- The code is compiled only with the iOS 27 SDK. Built with Xcode 26 or earlier, the pod is empty.

## How it was checked

Xcode 27.0 (27A266a) on macOS 27.0, iPhone 18 Pro simulator on iOS 27.0.

| App | Without the bridge | With it |
| --- | --- | --- |
| React Native 0.86.3, community template | Fails to launch | Renders. `Linking.getInitialURL()` returns the link that launched the app, a link opened while it runs arrives as a `url` event, and `AppState` reports `background` and `active` |
| Expo SDK 56, default template with Expo Router (React Native 0.85.3) | Fails to launch | Renders. A link that launches the app opens the linked route, and a link opened while it runs navigates |
| Expo SDK 54, blank template (React Native 0.81.5) | Fails to launch | Renders |
| Minimal UIKit app on the app-based life cycle | Fails to launch | Launches. The window follows the scene into landscape. Links reach `application:openURL:options:` whether they launch the app or arrive while it runs, and the app delegate's foreground and background methods run as they did before |
| Apps that declare a scene, in code or in `Info.plist` | | Launch with their own scene delegate. The bridge stays out |

The last two rows are automated: `npm test` builds those apps and runs them in the booted
simulator, and CI runs it on GitHub's Xcode 27 image. Five copies of the bridge, each with one
behaviour removed, each fail the check for it.

## Limits

- Tested in the simulator only, not yet on a device.
- One window. iPad multiwindow needs real scene support.
- Universal links, Handoff and home-screen quick actions go through the same forwarding as
  custom-scheme links, but were not tested.
- Tested on iOS only. Mac Catalyst builds are untested, and tvOS and visionOS are not supported.
- It handles the launch requirement. Other errors when building with Xcode 27 are out of scope.
- Not affiliated with Apple, Meta or Expo.

## License

MIT
