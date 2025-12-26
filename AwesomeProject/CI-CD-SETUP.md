# CI/CD Setup Guide for React Native

This document outlines the CI/CD configuration for building Android and iOS applications for different environments.

## Overview

### Development Branch
- **Android**: Builds APK (unsigned/debug signed)
- **iOS**: Builds Archive and IPA for development distribution

### Production Branch
- **Android**: Builds AAB (signed) and APK (signed) for Play Store
- **iOS**: Builds Archive and IPA for App Store distribution (with optional TestFlight upload)

## Workflows Created

1. [android-development.yml](.github/workflows/android-development.yml) - Android development builds
2. [android-production.yml](.github/workflows/android-production.yml) - Android production builds
3. [ios-development.yml](.github/workflows/ios-development.yml) - iOS development builds
4. [ios-production.yml](.github/workflows/ios-production.yml) - iOS production builds

## Required Secrets

### Android Production Secrets

Add these secrets to your GitHub repository (Settings > Secrets and variables > Actions):

1. **ANDROID_KEYSTORE_BASE64**
   - Your release keystore file encoded in base64
   - Generate: `base64 -i your-release-key.keystore | pbcopy` (macOS) or `base64 -w 0 your-release-key.keystore` (Linux)

2. **ANDROID_KEYSTORE_PASSWORD**
   - Password for the keystore file

3. **ANDROID_KEY_ALIAS**
   - Alias name for the signing key

4. **ANDROID_KEY_PASSWORD**
   - Password for the signing key

### iOS Production Secrets

1. **IOS_CERTIFICATE_BASE64**
   - Your Apple distribution certificate (.p12) encoded in base64
   - Generate: `base64 -i certificate.p12 | pbcopy` (macOS) or `base64 -w 0 certificate.p12` (Linux)

2. **IOS_CERTIFICATE_PASSWORD**
   - Password for the .p12 certificate

3. **IOS_KEYCHAIN_PASSWORD**
   - Any secure password for the temporary keychain (e.g., generate random string)

4. **IOS_PROVISIONING_PROFILE_BASE64**
   - Your provisioning profile (.mobileprovision) encoded in base64
   - Generate: `base64 -i profile.mobileprovision | pbcopy` (macOS)

5. **IOS_CODE_SIGN_IDENTITY**
   - Your code signing identity (e.g., "iPhone Distribution: Your Company Name (TEAM_ID)")

6. **IOS_PROVISIONING_PROFILE_SPECIFIER**
   - Name of your provisioning profile

### Optional: TestFlight Upload Secrets

7. **APP_STORE_CONNECT_API_KEY_ID**
   - App Store Connect API Key ID

8. **APP_STORE_CONNECT_API_ISSUER_ID**
   - App Store Connect API Issuer ID

9. **APP_STORE_CONNECT_API_KEY**
   - App Store Connect API Key (base64 encoded .p8 file)

## Setup Instructions

### 1. Android Production Setup

#### Generate Release Keystore

```bash
cd android/app
keytool -genkey -v -keystore release.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
```

Follow the prompts to set passwords and details.

#### Add Secrets to GitHub

1. Encode your keystore:
   ```bash
   base64 -i android/app/release.keystore | pbcopy
   ```
2. Go to GitHub repository → Settings → Secrets and variables → Actions → New repository secret
3. Add all Android secrets listed above

### 2. iOS Production Setup

#### Export Certificate from Keychain

1. Open Keychain Access on macOS
2. Find your distribution certificate
3. Right-click → Export
4. Save as .p12 file with a password

#### Get Provisioning Profile

1. Go to Apple Developer Portal
2. Download your App Store provisioning profile
3. Save as .mobileprovision file

#### Encode and Add to GitHub

```bash
# Encode certificate
base64 -i certificate.p12 | pbcopy

# Encode provisioning profile
base64 -i profile.mobileprovision | pbcopy
```

Add all iOS secrets to GitHub repository secrets.

#### Update Export Options

Edit [ios/ExportOptionsDevelopment.plist](ios/ExportOptionsDevelopment.plist) and [ios/ExportOptionsProduction.plist](ios/ExportOptionsProduction.plist):

- Replace `YOUR_TEAM_ID` with your Apple Team ID
- Replace `YOUR_DEVELOPMENT_PROVISIONING_PROFILE_NAME` with your development profile name
- Replace `YOUR_APPSTORE_PROVISIONING_PROFILE_NAME` with your App Store profile name
- Update `com.awesomeproject` with your actual bundle identifier

### 3. Branch Configuration

The workflows are configured to run on:
- **Development**: `development` branch
- **Production**: `production` or `main` branch

Create these branches:
```bash
git checkout -b development
git push -u origin development

git checkout -b production
git push -u origin production
```

## Build Artifacts

### Android
- **Development**: APK available in GitHub Actions artifacts (30 days retention)
- **Production**: AAB and APK available in GitHub Actions artifacts (60 days retention)

### iOS
- **Development**: Archive and IPA available in GitHub Actions artifacts (30 days retention)
- **Production**: Archive and IPA available in GitHub Actions artifacts (60 days retention)

## Releases

Each successful build creates a GitHub Release:
- **Development**: Pre-release tagged as `dev-v1.0.{run_number}` or `ios-dev-v1.0.{run_number}`
- **Production**: Release tagged as `prod-v1.0.{run_number}` or `ios-prod-v1.0.{run_number}`

## Troubleshooting

### Android Build Fails

1. Check Java version (requires JDK 17)
2. Verify Gradle wrapper is executable
3. Check signing configuration in secrets

### iOS Build Fails

1. Verify certificate and provisioning profile are valid
2. Check code signing identity matches certificate
3. Ensure bundle identifier matches in all places
4. Verify Xcode version compatibility

### Common Issues

**"Keystore file not found"**
- Ensure ANDROID_KEYSTORE_BASE64 is properly encoded and added to secrets

**"Code signing failed"**
- Verify certificate is for distribution (not development)
- Check provisioning profile includes the device/app ID

**"No such file or directory: ExportOptions.plist"**
- Ensure ExportOptions files are committed to repository
- Update with correct team ID and profile names

## Additional Configuration

### Versioning

Update version codes/names in:
- Android: [android/app/build.gradle](android/app/build.gradle) (versionCode, versionName)
- iOS: Update in Xcode project settings

### Environment Variables

To add environment-specific variables:
1. Add to GitHub secrets
2. Use in workflow files with `${{ secrets.YOUR_SECRET }}`
3. Pass to build commands as needed

### Custom Build Scripts

Add scripts to [package.json](package.json):
```json
{
  "scripts": {
    "build:android:dev": "cd android && ./gradlew assembleRelease",
    "build:android:prod": "cd android && ./gradlew bundleRelease",
    "build:ios": "react-native run-ios --configuration Release"
  }
}
```

## Testing the Workflow

### Test Development Build

```bash
git checkout development
# Make a change
git add .
git commit -m "Test development build"
git push origin development
```

### Test Production Build

```bash
git checkout production
# Make a change
git add .
git commit -m "Test production build"
git push origin production
```

Check the Actions tab in your GitHub repository to monitor build progress.

## Notes

- Development builds use debug keystore (no additional setup needed)
- Production builds require proper code signing
- iOS builds require macOS runners (more expensive on GitHub Actions)
- Consider using self-hosted runners for cost savings
- AAB format is required for Google Play Store (production)
- APK format is for direct distribution or testing

## Resources

- [React Native Documentation](https://reactnative.dev/docs/signed-apk-android)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Fastlane for React Native](https://docs.fastlane.tools/getting-started/cross-platform/react-native/)
