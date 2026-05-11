# ntfy

A Flutter plugin to natively subscribe to [ntfy](https://ntfy.sh/) (Push notifications made easy) topics on Android using a persistent Foreground Service.

This plugin allows your Flutter application to connect to an ntfy server's JSON stream, maintain a background connection using an Android Foreground Service, and stream incoming notifications directly into your Dart code.

## Features

- **Native Android Foreground Service**: Keeps the connection alive even when your app is in the background or screen is off.
- **Server-Sent Events (SSE) JSON Stream**: Efficiently listens to incoming ntfy notifications.
- **Authentication Support**: Native support for passing access tokens or basic authentication credentials to private ntfy instances.
- **Flutter EventChannel**: Streams received messages back to your Flutter app in real-time.

## Platform Support

Currently, this plugin only supports **Android**.

## Getting Started

### 1. Android Setup

Since this plugin relies on an Android Foreground Service, you need to ensure you request the necessary permissions in your app before starting the service (especially on Android 13+).

In your `android/app/src/main/AndroidManifest.xml`, the plugin automatically requests:
- `android.permission.INTERNET`
- `android.permission.WAKE_LOCK`
- `android.permission.FOREGROUND_SERVICE`
- `android.permission.POST_NOTIFICATIONS`

### 2. Requesting Permissions in Flutter

Before calling the `subscribe` method, make sure you request notification permissions from the user. You can use the `permission_handler` package for this:

```dart
import 'package:permission_handler/permission_handler.dart';

Future<void> requestPermissions() async {
  if (await Permission.notification.isDenied) {
    await Permission.notification.request();
  }
}
```

### 3. Usage

Import the plugin and subscribe to a topic. You can listen to the incoming messages using the `messages` stream.

```dart
import 'package:ntfy/ntfy.dart';

class NtfyExample {
  final _ntfyPlugin = Ntfy();
  StreamSubscription<String>? _messageSubscription;

  void startListening() async {
    // 1. Start listening to the message stream
    _messageSubscription = _ntfyPlugin.messages.listen((String messageJson) {
      print("Received new ntfy message: $messageJson");
      // Note: The message is a JSON string. You can use jsonDecode(messageJson) to parse it.
    });

    // 2. Subscribe and start the Android Foreground Service
    // Connects to: https://ntfy.sh/my_awesome_topic
    await _ntfyPlugin.subscribe('https://ntfy.sh', 'my_awesome_topic');
  }

  void stopListening() {
    _messageSubscription?.cancel();
    _ntfyPlugin.unsubscribe();
  }
}
```

### Authentication

If you are using a self-hosted ntfy server with access control, you can pass an `auth` parameter to the `subscribe` method.

#### Using an Access Token (Bearer Auth)
```dart
await _ntfyPlugin.subscribe(
  'https://ntfy.example.com', 
  'secret_topic',
  auth: 'Bearer tk_your_access_token_here',
);
```

#### Using Username and Password (Basic Auth)
```dart
import 'dart:convert';

final String credentials = base64Encode(utf8.encode('username:password'));

await _ntfyPlugin.subscribe(
  'https://ntfy.example.com', 
  'secret_topic',
  auth: 'Basic $credentials',
);
```

## How it Works

When you call `subscribe()`, the plugin starts an Android Foreground Service named `NtfyForegroundService`. This service holds a partial wake lock and opens an `HttpURLConnection` to the ntfy `/json` stream endpoint. Whenever a new JSON message is received, it is broadcasted back to the Flutter Engine over a standard `EventChannel`.
# ntfy
# ntfy
