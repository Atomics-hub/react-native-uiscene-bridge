# Changelog

## 0.1.0

First release.

- Adopts the UIScene life cycle for apps that have not, so they launch when built with the iOS 27
  SDK (Xcode 27) instead of failing with "UIScene life cycle is required for apps built with this
  SDK". No code changes.
- Puts the window the app delegate created into the scene, and passes links, user activities,
  quick actions and foreground and background transitions back to the app delegate methods that
  used to receive them.
- Answers React Native's `Linking.getInitialURL()` with the link that launched the app.
- Does nothing in apps that declare a scene, and compiles to nothing with SDKs before iOS 27.
- Checked in the iOS 27 simulator with React Native 0.86.3, Expo SDK 56 with Expo Router and Expo
  SDK 54.
