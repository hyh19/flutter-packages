# XFile IO Implementation

## Overview

The `XFile` class in `io.dart` provides a concrete implementation of the `XFileBase` abstract class, specifically designed for Dart's `dart:io` platform (typically used in Flutter mobile and desktop applications). This implementation bridges the gap between platform-specific file operations and the cross-platform `XFile` interface.

```dart 13:14:lib/src/types/io.dart
/// A CrossFile backed by a dart:io File.
class XFile extends XFileBase {
```

The class serves as a wrapper around Dart's `File` class, providing a consistent API that works across different platforms while maintaining the ability to perform native file system operations.

## Key Characteristics

- **Platform-specific**: Uses `dart:io` for direct file system access
- **Dual storage modes**: Can represent either physical files on disk or in-memory byte data
- **Cross-platform compatibility**: Implements the same interface as web-based XFile implementations
- **Lazy evaluation**: File metadata is computed only when needed

## Constructor Options

### Primary Constructor: XFile

Creates an XFile instance backed by an existing file on the file system.

```dart 27:39:lib/src/types/io.dart
XFile(
  String path, {
  String? mimeType,
  String? name,
  int? length,
  Uint8List? bytes,
  DateTime? lastModified,
}) : _mimeType = mimeType,
     _file = File(path),
     _bytes = null,
     _lastModified = lastModified,
     super(path);
```

**Parameters:**

- `path`: The file system path to the file
- `mimeType`: Optional MIME type override
- `name`: Ignored (exists for web compatibility)
- `length`: Ignored (exists for web compatibility)
- `bytes`: Ignored (exists for web compatibility)
- `lastModified`: Optional last modified timestamp override

### Data Constructor: XFile.fromData

Creates an XFile instance from raw byte data, useful for handling files that exist only in memory.

```dart 45:61:lib/src/types/io.dart
XFile.fromData(
  Uint8List bytes, {
  String? mimeType,
  String? path,
  String? name,
  int? length,
  DateTime? lastModified,
}) : _mimeType = mimeType,
     _bytes = bytes,
     _file = File(path ?? ''),
     _length = length,
     _lastModified = lastModified,
     super(path) {
  if (length == null) {
    _length = bytes.length;
  }
}
```

**Parameters:**

- `bytes`: The raw file data as bytes
- `mimeType`: Optional MIME type
- `path`: Optional file path (defaults to empty string)
- `name`: Ignored (exists for web compatibility)
- `length`: Optional length override (defaults to bytes.length)
- `lastModified`: Optional last modified timestamp

## Core Properties

### File Path and Name

```dart 94:98:lib/src/types/io.dart
@override
String get path => _file.path;

@override
String get name => _file.path.split(Platform.pathSeparator).last;
```

- `path`: Returns the full file system path
- `name`: Extracts just the filename from the path using platform-appropriate separators

### MIME Type

```dart 91:92:lib/src/types/io.dart
@override
String? get mimeType => _mimeType;
```

Returns the MIME type if provided during construction, otherwise null.

## File Operations

### Reading File Content

#### readAsString()

```dart 108:116:lib/src/types/io.dart
@override
Future<String> readAsString({Encoding encoding = utf8}) {
  if (_bytes != null) {
    // TODO(kevmoo): Remove ignore and fix when the MIN Dart SDK is 3.3
    // ignore: unnecessary_non_null_assertion
    return Future<String>.value(String.fromCharCodes(_bytes!));
  }
  return _file.readAsString(encoding: encoding);
}
```

Reads the entire file content as a string. For in-memory files, converts bytes directly; for disk files, uses the underlying File's readAsString method.

#### readAsBytes()

```dart 118:124:lib/src/types/io.dart
@override
Future<Uint8List> readAsBytes() {
  if (_bytes != null) {
    return Future<Uint8List>.value(_bytes);
  }
  return _file.readAsBytes();
}
```

Returns the file content as bytes. Similar to readAsString, it handles both in-memory and disk-based files appropriately.

### Streaming Access

#### openRead()

```dart 131:140:lib/src/types/io.dart
@override
Stream<Uint8List> openRead([int? start, int? end]) {
  if (_bytes != null) {
    return _getBytes(start, end);
  } else {
    return _file
        .openRead(start ?? 0, end)
        .map((List<int> chunk) => Uint8List.fromList(chunk));
  }
}
```

Provides streaming access to file content, useful for large files. For in-memory files, it uses a custom implementation; for disk files, it leverages the File's openRead method.

```dart 126:129:lib/src/types/io.dart
Stream<Uint8List> _getBytes(int? start, int? end) async* {
  final Uint8List bytes = _bytes!;
  yield bytes.sublist(start ?? 0, end ?? bytes.length);
}
```

The helper method for streaming in-memory byte data.

### File Metadata

#### length()

```dart 100:106:lib/src/types/io.dart
@override
Future<int> length() {
  if (_length != null) {
    return Future<int>.value(_length);
  }
  return _file.length();
}
```

Returns the file size in bytes. Uses cached length for in-memory files or queries the file system for disk files.

#### lastModified()

```dart 70:77:lib/src/types/io.dart
@override
Future<DateTime> lastModified() {
  if (_lastModified != null) {
    return Future<DateTime>.value(_lastModified);
  }
  // ignore: avoid_slow_async_io
  return _file.lastModified();
}
```

Returns the last modified timestamp. Uses provided timestamp or queries the file system.

### Saving Files

#### saveTo()

```dart 79:89:lib/src/types/io.dart
@override
Future<void> saveTo(String path) async {
  if (_bytes == null) {
    await _file.copy(path);
  } else {
    final File fileToSave = File(path);
    // TODO(kevmoo): Remove ignore and fix when the MIN Dart SDK is 3.3
    // ignore: unnecessary_non_null_assertion
    await fileToSave.writeAsBytes(_bytes!);
  }
}
```

Saves the file content to a specified path. For disk-backed files, copies the existing file; for memory-backed files, writes the byte data to disk.

## Implementation Details

### Storage Strategy

The class employs a dual-storage approach to handle different file sources:

- **Disk files**: When created with `XFile(path)`, the file exists on the file system
- **Memory files**: When created with `XFile.fromData(bytes)`, the data exists only in memory

This design allows the same API to work with files from various sources while optimizing performance based on the storage type.

### Parameter Compatibility

Many constructor parameters are marked as "ignored" to maintain API compatibility with web implementations:

```dart 16:26:lib/src/types/io.dart
/// [bytes] is ignored; the parameter exists only to match the web version of
/// the constructor. To construct a dart:io XFile from bytes, use
/// [XFile.fromData].
///
/// [name] is ignored; the parameter exists only to match the web version of
/// the constructor.
///
/// [length] is ignored; the parameter exists only to match the web version of
/// the constructor.
```

This ensures that code written for one platform can be easily adapted to another.

### Error Handling and Validation

The implementation includes minimal validation, relying on the underlying `dart:io` File class for most error handling. The `XFile.fromData` constructor does perform basic length calculation when not explicitly provided.

### Performance Considerations

- **Lazy evaluation**: File metadata is only computed when requested
- **Streaming support**: Large files can be processed without loading entirely into memory
- **Direct file operations**: For disk files, operations are delegated directly to the OS file system APIs

## Usage Patterns

### Working with Existing Files

```dart
final file = XFile('/path/to/document.pdf', mimeType: 'application/pdf');
final content = await file.readAsString();
final size = await file.length();
```

### Working with Generated Content

```dart
final data = Uint8List.fromList([1, 2, 3, 4, 5]);
final file = XFile.fromData(data, mimeType: 'application/octet-stream');
await file.saveTo('/path/to/output.bin');
```

### Streaming Large Files

```dart
final file = XFile('/path/to/large-video.mp4');
final stream = file.openRead();
await for (final chunk in stream) {
  // Process chunk
}
```

This implementation provides a robust foundation for file handling in Flutter applications, balancing cross-platform compatibility with native performance.
