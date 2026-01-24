# XFileBase - Cross-Platform File Interface

## Overview

The `XFileBase` class serves as the foundational abstract interface for the cross_file package, providing a unified way to handle file operations across different platforms, particularly web and mobile environments.

## Class Purpose

`XFileBase` is designed to be a very limited subset of `dart:io`'s `File` class, offering familiar file manipulation methods while ensuring cross-platform compatibility. It acts as a container that wraps file paths and, on platforms like web, the actual file bytes.

```dart 16:16:lib/src/types/base.dart
abstract class XFileBase {
```

## Constructor

The class takes an optional `path` parameter in its constructor, though it's marked as unused in this base implementation:

```dart 17:19:lib/src/types/base.dart
  /// Construct a CrossFile
  // ignore: avoid_unused_constructor_parameters
  XFileBase(String? path);
```

## Core Methods

### File Path Access

#### `path` - File Path Getter

Returns the filesystem path of the selected file. The documentation explicitly warns against using this for actual file access, recommending it only for display purposes or backwards compatibility:

```dart 26:37:lib/src/types/base.dart
  /// Get the path of the picked file.
  ///
  /// This should only be used as a backwards-compatibility clutch
  /// for mobile apps, or cosmetic reasons only (to show the user
  /// the path they've picked).
  ///
  /// Accessing the data contained in the picked file by its path
  /// is platform-dependant (and won't work on web), so use the
  /// byte getters in the CrossFile instance instead.
  String get path {
    throw UnimplementedError('.path has not been implemented.');
  }
```

**Key Warning**: Direct path-based file access won't work on web platforms.

#### `name` - File Name Getter

Provides the filename as selected by the user. For non-web implementations, this represents the last segment of the filesystem path:

```dart 39:46:lib/src/types/base.dart
  /// The name of the file as it was selected by the user in their device.
  ///
  /// For non-web implementation, this represents the last part of the filesystem path.
  ///
  /// Use only for cosmetic reasons, do not try to use this as a path.
  String get name {
    throw UnimplementedError('.name has not been implemented.');
  }
```

### MIME Type Support

#### `mimeType` - MIME Type Getter

Returns the MIME type of the file, which is particularly important for web platforms where file type detection may be necessary:

```dart 48:51:lib/src/types/base.dart
  /// For web, it may be necessary for a file to know its MIME type.
  String? get mimeType {
    throw UnimplementedError('.mimeType has not been implemented.');
  }
```

## Content Reading Methods

### `length()` - File Size

Returns the file size in bytes as a Future:

```dart 53:56:lib/src/types/base.dart
  /// Get the length of the file. Returns a `Future<int>` that completes with the length in bytes.
  Future<int> length() {
    throw UnimplementedError('.length() has not been implemented.');
  }
```

### `readAsString()` - Read as String

Reads the entire file contents as a string with optional encoding (defaults to UTF-8):

```dart 58:65:lib/src/types/base.dart
  /// Asynchronously read the entire file contents as a string using the given [Encoding].
  ///
  /// By default, `encoding` is [utf8].
  ///
  /// Throws Exception if the operation fails.
  Future<String> readAsString({Encoding encoding = utf8}) {
    throw UnimplementedError('readAsString() has not been implemented.');
  }
```

### `readAsBytes()` - Read as Bytes

Reads the entire file contents as a `Uint8List`:

```dart 67:72:lib/src/types/base.dart
  /// Asynchronously read the entire file contents as a list of bytes.
  ///
  /// Throws Exception if the operation fails.
  Future<Uint8List> readAsBytes() {
    throw UnimplementedError('readAsBytes() has not been implemented.');
  }
```

### `openRead()` - Streaming Read

Creates a stream for reading file contents, supporting optional start and end byte offsets:

```dart 74:83:lib/src/types/base.dart
  /// Create a new independent [Stream] for the contents of this file.
  ///
  /// If `start` is present, the file will be read from byte-offset `start`. Otherwise from the beginning (index 0).
  ///
  /// If `end` is present, only up to byte-index `end` will be read. Otherwise, until end of file.
  ///
  /// In order to make sure that system resources are freed, the stream must be read to completion or the subscription on the stream must be cancelled.
  Stream<Uint8List> openRead([int? start, int? end]) {
    throw UnimplementedError('openRead() has not been implemented.');
  }
```

**Important**: The stream must be fully consumed or the subscription cancelled to prevent resource leaks.

## File Metadata

### `lastModified()` - Last Modified Time

Returns the file's last modification timestamp:

```dart 85:88:lib/src/types/base.dart
  /// Get the last-modified time for the CrossFile
  Future<DateTime> lastModified() {
    throw UnimplementedError('lastModified() has not been implemented.');
  }
```

## File Writing

### `saveTo()` - Save File

Saves the file to a specified path:

```dart 21:24:lib/src/types/base.dart
  /// Save the CrossFile at the indicated file path.
  Future<void> saveTo(String path) {
    throw UnimplementedError('saveTo has not been implemented.');
  }
```

## Implementation Pattern

All methods in this base class throw `UnimplementedError`, indicating that concrete implementations must override these methods. This is a common pattern for abstract interfaces in Dart.

## Platform Considerations

The interface is specifically designed with cross-platform compatibility in mind:

- **Web**: Cannot rely on filesystem paths, must use byte data directly
- **Mobile**: Can use filesystem paths but should prefer the interface methods for consistency
- **Desktop**: Similar to mobile platforms

The emphasis on using byte-based methods (`readAsBytes()`, `openRead()`) rather than path-based access ensures consistent behavior across all platforms.

## Usage Philosophy

The class documentation emphasizes that this is a "very limited subset" of `dart:io`'s `File`, suggesting that developers familiar with Dart's standard file I/O will find the API familiar. The focus is on essential file operations needed for cross-platform file picking and handling scenarios.
