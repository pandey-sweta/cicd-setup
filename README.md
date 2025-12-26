# AwesomeProject - React Native Mobile Application

A React Native application with fully automated CI/CD pipelines for building and deploying Android and iOS applications.

## Features

- Cross-platform mobile application (iOS & Android)
- Automated CI/CD workflows using GitHub Actions
- Environment-specific builds (Development & Production)
- Automated artifact generation and release management

## Build Configurations

### Development Branch
- **Android**: Generates APK for testing
- **iOS**: Generates Archive and IPA for development distribution
- Auto-publishes as pre-release on GitHub

### Production Branch
- **Android**: Generates AAB (for Play Store) and APK (signed)
- **iOS**: Generates Archive and IPA with optional TestFlight upload
- Auto-publishes as official release on GitHub

## Tech Stack

- React Native 0.83.1
- React 19.2.0
- TypeScript
- GitHub Actions for CI/CD
- Gradle for Android builds
- Xcode for iOS builds

## Getting Started

See [CI-CD-SETUP.md](CI-CD-SETUP.md) for detailed setup instructions.
