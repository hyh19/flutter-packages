# XFile Interface Definition

## Overview

The `interface.dart` file defines the core `XFile` class and its associated testing infrastructure for the cross-file package. This file serves as the platform-agnostic interface that provides a unified abstraction for file handling across different platforms (web, mobile, desktop).

## XFile Class

The `XFile` class is the main cross-platform file abstraction that extends `XFileBase`. It provides a simplified interface for working with file data regardless of the underlying platform implementation.

### Primary Constructor

```dart 24:36:lib/src/types/interface.dart
XFile(
  String super.path, {
  String? mimeType,
  String? name,
  int? length,
  Uint8List? bytes,
  DateTime? lastModified,
  @visibleForTesting CrossFileTestOverrides? overrides,
}) {
  throw UnimplementedError(
    'CrossFile is not available in your current platform.',
  );
}
```

**Parameters:**

- `path`: The platform-dependent file path or identifier
- `mimeType`: Optional MIME type of the file
- `name`: Optional display name (useful when the actual path differs from user-visible name, particularly on web)
- `length`: Optional file size in bytes
- `bytes`: Optional pre-loaded file content as bytes
- `lastModified`: Optional last modification timestamp
- `overrides`: Testing overrides for platform-specific behavior

**Implementation Note:** This constructor immediately throws an `UnimplementedError`, indicating that platform-specific implementations must override this behavior. Each platform (web, io, etc.) provides its own concrete implementation.

### fromData Constructor

```dart 42:54:lib/src/types/interface.dart
XFile.fromData(
  Uint8List bytes, {
  String? mimeType,
  String? name,
  int? length,
  DateTime? lastModified,
  String? path,
  @visibleForTesting CrossFileTestOverrides? overrides,
}) : super(path) {
  throw UnimplementedError(
    'CrossFile is not available in your current platform.',
  );
}
```

**Parameters:**

- `bytes`: The file content as a byte array (required)
- `mimeType`: Optional MIME type
- `name`: Optional display name
- `length`: Optional file size (can be inferred from bytes if not provided)
- `lastModified`: Optional modification timestamp
- `path`: Optional path (ignored on web platforms when bytes are provided)
- `overrides`: Testing overrides

**Key Difference:** Unlike the primary constructor, this factory constructor requires actual file data (`bytes`) and is designed for creating `XFile` instances from in-memory data rather than from a file path.

## Platform-Specific Behavior

### Web Platform Considerations

On web platforms, the `path` parameter in `fromData` is ignored when `bytes` are provided, as the underlying implementation uses a Blob URL as the effective path. This allows for efficient memory-based file handling without requiring actual file system access.

### Error Handling Strategy

Both constructors throw `UnimplementedError` with the message "CrossFile is not available in your current platform." This design ensures that:

1. Platform-specific implementations must provide concrete behavior
2. Attempting to use the base interface directly results in clear error messaging
3. The API contract is maintained across all platforms

## CrossFileTestOverrides Class

```dart 57:65:lib/src/types/interface.dart
/// Overrides some functions of CrossFile for testing purposes
@visibleForTesting
class CrossFileTestOverrides {
  /// Default constructor for overrides
  CrossFileTestOverrides({required this.createAnchorElement});

  /// For overriding the creation of the file input element.
  dynamic Function(String href, String suggestedName) createAnchorElement;
}
```

The `CrossFileTestOverrides` class provides testing infrastructure specifically for web platform file operations. It allows tests to override the creation of anchor elements used for file downloads, enabling controlled testing of download functionality without actual browser interactions.

**Usage Context:** This class is marked with `@visibleForTesting`, indicating it's only accessible during testing and not part of the public API.

## Architecture Notes

### Inheritance Pattern

The `XFile` class extends `XFileBase` (imported from `./base.dart`), suggesting a layered architecture where:

- `XFileBase` provides common functionality and abstract methods
- `XFile` defines the interface contract with concrete constructors
- Platform-specific implementations (like `XFileWeb`, `XFileIO`) provide the actual behavior

### Platform Abstraction Strategy

This interface follows Flutter's platform abstraction pattern:

1. Define platform-agnostic constructors that throw errors
2. Use conditional imports and factory constructors in platform-specific files
3. Ensure consistent API across all supported platforms

### Testing Design

The inclusion of `CrossFileTestOverrides` demonstrates thoughtful testing architecture, particularly for web platform features that interact with browser APIs. This allows for:

- Mocking browser-specific behaviors
- Testing download functionality without side effects
- Platform-specific test coverage

## Usage Implications

When working with this interface:

1. **Never instantiate directly** - Always use platform-specific factory methods or imports
2. **Handle platform differences** - Be aware that some parameters (like `path` in web contexts) may behave differently
3. **Test thoroughly** - Use the provided testing infrastructure for reliable cross-platform testing
4. **Check platform availability** - The error messages indicate when platform-specific implementations aren't available

This design enables seamless file handling across Flutter's supported platforms while maintaining clean separation between interface definition and platform-specific implementations.
