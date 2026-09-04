---
name: react-native-firebase-auth
description: Step-by-step guide to install, configure, and implement Firebase Authentication in modern React Native CLI projects (0.87+).
---

# React Native Firebase Authentication Setup Guide

This skill provides a complete, battle-tested, step-by-step guide for integrating **Firebase Authentication** into a **React Native CLI** project (React Native 0.87.x+, React 19+, modern Gradle & CocoaPods).

---

## Table of Contents
1. [Prerequisites & Compatibility](#1-prerequisites--compatibility)
2. [Step 1: Install Dependencies](#step-1-install-dependencies)
3. [Step 2: Firebase Console Configuration](#step-2-firebase-console-configuration)
4. [Step 3: Android Configuration](#step-3-android-configuration)
5. [Step 4: iOS Configuration](#step-4-ios-configuration)
6. [Step 5: Modular TypeScript Service Layer](#step-5-modular-typescript-service-layer)
7. [Step 6: UI & State Integration (Hook / Component Example)](#step-6-ui--state-integration-hook--component-example)
8. [Troubleshooting & Common Pitfalls](#troubleshooting--common-pitfalls)

---

## 1. Prerequisites & Compatibility

- **React Native**: 0.87.x or higher
- **React**: 19.x or higher
- **@react-native-firebase/app**: `^26.x.x` (or latest)
- **@react-native-firebase/auth**: `^26.x.x` (or latest)
- **Android**: Gradle 8.x/9.x, Android SDK 34/35/36+
- **iOS**: CocoaPods, iOS 15.1+ deployment target

---

## Step 1: Install Dependencies

Run the installation command using your package manager of choice in the root of your React Native project:

### Using Bun:
```bash
bun add @react-native-firebase/app @react-native-firebase/auth
```

### Using NPM / Yarn:
```bash
# npm
npm install @react-native-firebase/app @react-native-firebase/auth

# yarn
yarn add @react-native-firebase/app @react-native-firebase/auth
```

> **Optional Add-ons:** If you also need Firestore or Cloud Storage:
> ```bash
> bun add @react-native-firebase/firestore @react-native-firebase/storage
> ```

---

## Step 2: Firebase Console Configuration

1. Go to the [Firebase Console](https://console.firebase.google.com/).
2. Click **Add project** (or select your existing project).
3. In the left navigation sidebar, navigate to **Build > Authentication**.
4. Click **Get Started** and enable your desired Sign-in providers:
   - **Email/Password**: Enable "Email/Password".
   - **Anonymous**: Enable "Anonymous" (great for guest flows).
   - **Google / Apple**: Configure client IDs if required.
5. In **Project Settings (Gear icon) > General**:
   - Add an **Android App** (enter your package name, e.g., `com.yourapp`).
   - Download the `google-services.json` file.
   - Add an **iOS App** (enter your iOS Bundle ID, e.g., `com.yourapp`).
   - Download the `GoogleService-Info.plist` file.

---

## Step 3: Android Configuration

### 3.1 Place `google-services.json`
Place the downloaded `google-services.json` file inside the `android/app/` folder:
```
android/
  app/
    google-services.json   <-- Place here
    build.gradle
    src/
```

### 3.2 Update Root `android/build.gradle`
Open `android/build.gradle` and add the Google Services classpath inside `buildscript.dependencies`:

```groovy
buildscript {
    ext {
        buildToolsVersion = "37.0.0"
        minSdkVersion = 24
        compileSdkVersion = 37
        targetSdkVersion = 36
        kotlinVersion = "2.2.0"
    }
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath("com.android.tools.build:gradle")
        classpath("com.facebook.react:react-native-gradle-plugin")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin")
        // ADD THIS LINE:
        classpath("com.google.gms:google-services:4.4.2")
    }
}

apply plugin: "com.facebook.react.rootproject"
```

### 3.3 Update App-Level `android/app/build.gradle`
Open `android/app/build.gradle` and apply the Google Services plugin near the top:

```groovy
apply plugin: "com.android.application"
apply plugin: "org.jetbrains.kotlin.android"
apply plugin: "com.facebook.react"
// ADD THIS LINE:
apply plugin: "com.google.gms.google-services"

android {
    namespace "com.yourapp"
    defaultConfig {
        applicationId "com.yourapp"
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode 1
        versionName "1.0"
    }
    ...
}
```

### 3.4 Clean Android Build Cache
Run a clean build to ensure Gradle picks up the new plugin:
```bash
cd android && ./gradlew clean && cd ..
```

---

## Step 4: iOS Configuration

### 4.1 Add `GoogleService-Info.plist`
1. Open the iOS project in Xcode:
   ```bash
   open ios/YourProjectName.xcworkspace
   ```
2. Drag and drop `GoogleService-Info.plist` into the root of your Xcode project tree (inside the main project folder next to `Info.plist`).
3. **Important**: When prompted by Xcode, ensure:
   - **"Copy items if needed"** is checked.
   - Your app target is selected under **"Add to targets"**.

### 4.2 Install CocoaPods
Run pod install from your terminal:
```bash
cd ios
bundle exec pod install # or: pod install
cd ..
```

---

## Step 5: Modular TypeScript Service Layer

Create a dedicated, reusable Firebase auth service inside `src/services/firebase/auth.ts`.

### File: `src/services/firebase/auth.ts`
```typescript
import {
  getAuth,
  createUserWithEmailAndPassword,
  signInWithEmailAndPassword,
  signInAnonymously,
  signOut,
  sendPasswordResetEmail,
  updateProfile,
  onAuthStateChanged,
  User,
  Auth,
  Unsubscribe,
} from '@react-native-firebase/auth';

/**
 * Returns the default Firebase Auth instance
 */
export const getFirebaseAuth = (): Auth => {
  return getAuth();
};

export interface SignUpParams {
  email: string;
  password: string;
  displayName?: string;
}

export interface SignInParams {
  email: string;
  password: string;
}

/**
 * Sign up a new user with Email and Password
 */
export const signUpWithEmail = async ({
  email,
  password,
  displayName,
}: SignUpParams): Promise<User> => {
  const auth = getFirebaseAuth();
  const credential = await createUserWithEmailAndPassword(
    auth,
    email.trim(),
    password
  );
  if (displayName && credential.user) {
    await updateProfile(credential.user, { displayName: displayName.trim() });
  }
  return credential.user;
};

/**
 * Sign in existing user with Email and Password
 */
export const signInWithEmail = async ({
  email,
  password,
}: SignInParams): Promise<User> => {
  const auth = getFirebaseAuth();
  const credential = await signInWithEmailAndPassword(
    auth,
    email.trim(),
    password
  );
  return credential.user;
};

/**
 * Sign in user anonymously (Guest mode)
 */
export const signInUserAnonymously = async (): Promise<User> => {
  const auth = getFirebaseAuth();
  const credential = await signInAnonymously(auth);
  return credential.user;
};

/**
 * Sign out the current authenticated user
 */
export const signOutUser = async (): Promise<void> => {
  const auth = getFirebaseAuth();
  await signOut(auth);
};

/**
 * Send a password reset email
 */
export const sendPasswordReset = async (email: string): Promise<void> => {
  const auth = getFirebaseAuth();
  await sendPasswordResetEmail(auth, email.trim());
};

/**
 * Get current user synchronously
 */
export const getCurrentUser = (): User | null => {
  try {
    return getFirebaseAuth().currentUser;
  } catch {
    return null;
  }
};

/**
 * Listen to real-time auth state changes
 */
export const subscribeToAuthState = (
  callback: (user: User | null) => void
): Unsubscribe => {
  try {
    const auth = getFirebaseAuth();
    return onAuthStateChanged(auth, callback);
  } catch (error) {
    console.warn('Error subscribing to auth state:', error);
    return () => {};
  }
};

export type { User, Auth };
```

### File: `src/services/firebase/index.ts`
```typescript
import { getApp, getApps } from '@react-native-firebase/app';

export const getFirebaseApp = () => {
  try {
    const apps = getApps();
    return apps.length > 0 ? getApp() : null;
  } catch {
    return null;
  }
};

export * from './auth';
```

---

## Step 6: UI & State Integration (Hook / Component Example)

### Custom Hook: `src/features/auth/useAuth.ts`
```typescript
import { useEffect, useState } from 'react';
import {
  User,
  subscribeToAuthState,
  signInUserAnonymously,
  signOutUser,
} from '../../services/firebase';

export function useAuth() {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const unsubscribe = subscribeToAuthState((currentUser) => {
      setUser(currentUser);
      setLoading(false);
    });

    return () => unsubscribe();
  }, []);

  return {
    user,
    loading,
    isAuthenticated: !!user,
    signInAnonymously: signInUserAnonymously,
    signOut: signOutUser,
  };
}
```

---

## Troubleshooting & Common Pitfalls

### 1. `Default app has not been initialized`
- **Cause**: Missing `google-services.json` (Android) or `GoogleService-Info.plist` (iOS).
- **Fix**: Verify file placement and run a clean rebuild:
  ```bash
  # Android clean
  cd android && ./gradlew clean && cd .. && bun run android

  # iOS clean
  cd ios && rm -rf Pods Podfile.lock && pod install && cd .. && bun run ios
  ```

### 2. Package Name / ApplicationId Mismatch
- Ensure the package name in `android/app/build.gradle` (`namespace` or `applicationId`) matches the one specified in the Firebase Console during Android app creation.

### 3. D8 / Dexing Warnings in Modern Gradle (Android)
- Warnings like `D8: Invalid stack map table` can occur with newer Gradle versions (9.x). They do not break runtime authentication, but ensure you keep `com.google.gms:google-services` updated to version `4.4.2+`.

### 4. iOS Pod Install or Build Fails
- Ensure minimum iOS target in `ios/Podfile` is at least `15.1`.
- If using modular headers, ensure your Podfile has `use_frameworks! :linkage => :static` if required by other dependencies.
