# Changelog

## 0.1.1

Docs only. No code changes.

- Adds a "Do you need it?" table: React Native 0.87 or earlier and Expo SDK 57 or earlier need it
  with Xcode 27; React Native 0.88+ and Expo SDK 58+ on the scene delegate do not.
- Notes that React Native 0.88 shipped on 2026-10-15 and that Expo SDK 58 moves to it.

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
