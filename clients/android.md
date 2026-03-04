# Android App

Wake your devices from anywhere with the WakeLink Android app.

## Installation

### Google Play Store

[![Get it on Google Play](https://play.google.com/intl/en_us/badges/static/images/badges/en_badge_web_generic.png)](https://play.google.com/store/apps/details?id=io.wakelink)

### APK Download

Download from [GitHub Releases](https://github.com/wakelinkdev/wakelink-android/releases)

### Requirements

- Android 8.0 (API 26) or higher
- Internet connection

---

## Getting Started

### 1. Sign In

1. Open the app
2. Tap **Sign In**
3. Enter your email and password
4. Or tap **Create Account** if new

### 2. Add Your First Device

If you've already registered devices via CLI or web:

1. Your devices appear automatically after sign-in
2. Pull down to refresh the list

To add a new device:

1. Tap the **+** button
2. Enter device name
3. Copy the generated token
4. Use this token when configuring your ESP device

### 3. Wake a Device

1. Tap the device in the list
2. Tap the large **Wake** button
3. Wait for confirmation

---

## Features

### Device List

- See all your devices at a glance
- Online/offline status indicators
- Last seen timestamps
- Pull-to-refresh

### Quick Actions

- Tap device → Wake immediately
- Long-press → Access more options
- Swipe left → Quick wake
- Swipe right → View details

### Home Screen Widget

Add a widget for instant access:

1. Long-press home screen
2. Select **Widgets**
3. Find **WakeLink**
4. Choose widget size:
   - **Small**: Single device, one-tap wake
   - **Large**: Multiple devices list

### Notifications

Get notified about:

- Wake command results (success/failure)
- Device status changes
- Beta updates

Configure in **Settings → Notifications**

---

## App Settings

Access via the gear icon or **Settings** menu.

### Account

- **Email**: Your registered email
- **Sign Out**: Log out of the app
- **Delete Account**: Remove all data

### Server

- **API Server**: Default or custom server URL
- **Custom Server**: Enter your self-hosted server

### Appearance

- **Theme**: Light, Dark, System
- **Language**: English, Русский (auto-detected)

### Notifications

- **Wake Results**: Show notifications for results
- **Background Updates**: Check device status periodically
- **Sound**: Notification sound on/off

### Advanced

- **Timeout**: Command timeout (default 30s)
- **Debug Mode**: Show detailed logs

---

## Using Custom Server

Connect to your self-hosted WakeLink server:

1. Open **Settings**
2. Tap **Server**
3. Toggle **Custom Server**
4. Enter your server URL:
   ```
   https://api.your-domain.com
   ```
5. Tap **Test Connection**
6. If successful, tap **Save**

!!! note "HTTPS Required"
    Android requires HTTPS for network connections. Make sure your server has valid TLS certificates.

---

## Widgets

### Single Device Widget

- Shows one device
- Status indicator (online/offline)
- One tap to wake
- Updates every 15 minutes

### Multi-Device Widget

- Shows up to 4 devices
- Status indicators for each
- Tap any device to wake
- Compact view

### Configuring Widgets

1. Add widget to home screen
2. Select device(s) to display
3. Choose update frequency
4. Optionally enable auto-wake on tap

---

## Troubleshooting

### "Connection Failed"

1. Check internet connection
2. Verify server URL is correct
3. Try toggling WiFi/mobile data
4. Check if server is online: visit API URL in browser

### "Authentication Error"

1. Sign out and sign back in
2. Check if password changed
3. Verify account on web dashboard

### "Device Offline"

1. Check if ESP device is powered
2. Verify ESP has WiFi connectivity
3. Check ESP serial logs
4. Device may need restart

### App Crashes

1. Clear app cache: Settings → Apps → WakeLink → Clear Cache
2. Reinstall the app
3. Report issue on GitHub with crash logs

### Widget Not Updating

1. Check if battery optimization is disabled for WakeLink
2. Widgets update every 15 minutes minimum
3. Tap widget to force refresh

---

## Battery Optimization

For reliable background updates:

1. Go to **Settings → Apps → WakeLink**
2. Tap **Battery**
3. Select **Unrestricted** or **Don't optimize**

This ensures widgets and notifications work properly.

---

## Privacy

The WakeLink app:

- ✅ Stores credentials securely in Android Keystore
- ✅ Uses HTTPS for all communications
- ✅ Does not track usage or collect analytics
- ✅ Does not access contacts, camera, or other permissions
- ❌ Does not sell or share your data

Required permissions:

| Permission | Reason |
|------------|--------|
| Internet | Connect to WakeLink server |
| Foreground Service | Background status updates |

---

## Feedback & Support

- **Bug Reports**: [GitHub Issues](https://github.com/wakelinkdev/wakelink-android/issues)
- **Feature Requests**: [GitHub Discussions](https://github.com/wakelinkdev/wakelink-android/discussions)
- **Community**: [Discord](https://discord.gg/wakelink)
- **Email**: support@wakelink.io

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-02 | Complete rewrite, Material You design |
| 1.5.0 | 2025-10 | Widgets, notifications |
| 1.0.0 | 2025-06 | Initial release |

---

## Open Source

The WakeLink Android app is open source:

- **Repository**: [github.com/wakelinkdev/wakelink-android](https://github.com/wakelinkdev/wakelink-android)
- **License**: MIT
- **Contributions**: Welcome! See CONTRIBUTING.md
