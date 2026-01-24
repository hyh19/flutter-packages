# ImagePicker Class - Flutter Image Picker Plugin

## Overview

The `ImagePicker` class is the main entry point for the Flutter image_picker plugin, providing a unified API for selecting images and videos from device camera or gallery across different platforms (iOS, Android, Web, Windows, macOS, and Linux).

## Architecture

The class follows the platform interface pattern common in Flutter plugins:

```dart 24:26:image_picker/lib/image_picker.dart
/// The platform interface that drives this plugin
@visibleForTesting
static ImagePickerPlatform get platform => ImagePickerPlatform.instance;
```

This allows the plugin to use different implementations for different platforms while maintaining a consistent API.

## Core Methods

### Image Picking

#### `pickImage()` - Single Image Selection

```dart 72:93:image_picker/lib/image_picker.dart
Future<XFile?> pickImage({
  required ImageSource source,
  double? maxWidth,
  double? maxHeight,
  int? imageQuality,
  CameraDevice preferredCameraDevice = CameraDevice.rear,
  bool requestFullMetadata = true,
}) {
  final ImagePickerOptions imagePickerOptions =
      ImagePickerOptions.createAndValidate(
        maxWidth: maxWidth,
        maxHeight: maxHeight,
        imageQuality: imageQuality,
        preferredCameraDevice: preferredCameraDevice,
        requestFullMetadata: requestFullMetadata,
      );

  return platform.getImageFromSource(
    source: source,
    options: imagePickerOptions,
  );
}
```

**Parameters:**

- `source`: Required `ImageSource` (camera or gallery)
- `maxWidth`/`maxHeight`: Optional size constraints
- `imageQuality`: Compression quality (0-100, null = original)
- `preferredCameraDevice`: Camera selection (rear/front, ignored for gallery)
- `requestFullMetadata`: Whether to fetch complete metadata (may require additional permissions)

**Key Features:**

- Supports both camera and gallery sources
- Automatic image compression and resizing
- Platform-specific camera device selection
- Metadata retrieval control

#### `pickMultiImage()` - Multiple Image Selection

```dart 130:150:image_picker/lib/image_picker.dart
Future<List<XFile>> pickMultiImage({
  double? maxWidth,
  double? maxHeight,
  int? imageQuality,
  int? limit,
  bool requestFullMetadata = true,
}) {
  final ImageOptions imageOptions = ImageOptions.createAndValidate(
    maxWidth: maxWidth,
    maxHeight: maxHeight,
    imageQuality: imageQuality,
    requestFullMetadata: requestFullMetadata,
  );

  return platform.getMultiImageWithOptions(
    options: MultiImagePickerOptions.createAndValidate(
      imageOptions: imageOptions,
      limit: limit,
    ),
  );
}
```

**Parameters:**

- `limit`: Maximum number of images to select
- Other parameters same as `pickImage()`

**Platform Notes:**

- Not supported on iOS versions lower than 14
- The `limit` parameter may be ignored on some platforms

### Media Picking (Images and Videos)

#### `pickMedia()` - Single Media Item

```dart 187:206:image_picker/lib/image_picker.dart
Future<XFile?> pickMedia({
  double? maxWidth,
  double? maxHeight,
  int? imageQuality,
  bool requestFullMetadata = true,
}) async {
  final List<XFile> listMedia = await platform.getMedia(
    options: MediaOptions.createAndValidate(
      imageOptions: ImageOptions.createAndValidate(
        maxHeight: maxHeight,
        maxWidth: maxWidth,
        imageQuality: imageQuality,
        requestFullMetadata: requestFullMetadata,
      ),
      allowMultiple: false,
    ),
  );

  return listMedia.isNotEmpty ? listMedia.first : null;
}
```

**Key Characteristics:**

- Gallery-only source (no camera option)
- Supports both images and videos
- Returns single item or null
- iOS 14+ minimum requirement

#### `pickMultipleMedia()` - Multiple Media Items

```dart 246:265:image_picker/lib/image_picker.dart
Future<List<XFile>> pickMultipleMedia({
  double? maxWidth,
  double? maxHeight,
  int? imageQuality,
  int? limit,
  bool requestFullMetadata = true,
}) {
  return platform.getMedia(
    options: MediaOptions.createAndValidate(
      allowMultiple: true,
      imageOptions: ImageOptions.createAndValidate(
        maxHeight: maxHeight,
        maxWidth: maxWidth,
        imageQuality: imageQuality,
        requestFullMetadata: requestFullMetadata,
      ),
      limit: limit,
    ),
  );
}
```

### Video Picking

#### `pickVideo()` - Single Video Selection

```dart 289:299:image_picker/lib/image_picker.dart
Future<XFile?> pickVideo({
  required ImageSource source,
  CameraDevice preferredCameraDevice = CameraDevice.rear,
  Duration? maxDuration,
}) {
  return platform.getVideo(
    source: source,
    preferredCameraDevice: preferredCameraDevice,
    maxDuration: maxDuration,
  );
}
```

**Parameters:**

- `source`: Camera or gallery
- `preferredCameraDevice`: Camera selection
- `maxDuration`: Maximum video length (null = unlimited)

#### `pickMultiVideo()` - Multiple Video Selection

```dart 317:321:image_picker/lib/image_picker.dart
Future<List<XFile>> pickMultiVideo({Duration? maxDuration, int? limit}) {
  return platform.getMultiVideoWithOptions(
    options: MultiVideoPickerOptions(maxDuration: maxDuration, limit: limit),
  );
}
```

**Notes:**

- Gallery-only for multiple selection
- Both `maxDuration` and `limit` may be ignored on some platforms

## Error Handling and Recovery

### `retrieveLostData()` - Android Activity Recovery

```dart 337:339:image_picker/lib/image_picker.dart
Future<LostDataResponse> retrieveLostData() {
  return platform.getLostData();
}
```

**Purpose:**

- Recovers lost data when Android MainActivity is destroyed during image/video picking
- Critical for Android apps that may be interrupted by system events

**Usage Pattern:**

```dart
// In app initialization or resume handler
final LostDataResponse response = await picker.retrieveLostData();
if (response.file != null) {
  // Process the recovered file
}
```

## Platform Support Checking

### `supportsImageSource()` - Capability Detection

```dart 345:347:image_picker/lib/image_picker.dart
bool supportsImageSource(ImageSource source) {
  return platform.supportsImageSource(source);
}
```

**Purpose:**

- Checks if the current platform supports a specific image source
- Prevents runtime errors from unsupported operations

## Key Concepts

### XFile - Cross-Platform File Abstraction

All methods return `XFile` objects, which provide:

- Cross-platform file path handling
- Metadata access (name, size, MIME type)
- File reading capabilities
- Session-based usage (don't persist paths)

### Image Processing Options

**Compression and Resizing:**

- `maxWidth`/`maxHeight`: Size constraints
- `imageQuality`: JPEG compression (0-100)
- Platform-dependent support for formats (JPEG, PNG, WebP, HEIC)

**Platform-Specific Behavior:**

- iOS: Supports HEIC, limited compression for non-JPEG
- Android: Variable HEIC support, broader compression options
- Web: Limited to browser APIs

### Permission Handling

**Automatic Permissions:**

- Camera/gallery access requested automatically
- `requestFullMetadata=true` may trigger additional permission prompts
- iOS: May request "Photo Library Usage" permission

**Error Types:**

- `PlatformException` for permission denials, hardware unavailability
- Activity allocation failures (Android)
- Temporary file creation issues (iOS)

### Session-Based Design

**Important Warning:**
All returned file paths are temporary and should only be used within the current app session. The underlying files may be deleted by the system after app restart.

## Platform-Specific Considerations

### iOS

- HEIC format support
- iOS 14+ required for multi-selection methods
- Photo Library permission handling
- Temporary file management

### Android

- Activity lifecycle management
- Camera intent limitations
- HEIC support varies by Android version
- Lost data recovery mechanism

### Web

- Browser API limitations
- File input restrictions
- No camera device selection

### Desktop (Windows/macOS/Linux)

- Native file picker integration
- Limited camera support
- Platform-specific UI behavior

## Usage Patterns

### Basic Image Picking

```dart
final ImagePicker picker = ImagePicker();

final XFile? image = await picker.pickImage(source: ImageSource.gallery);
if (image != null) {
  // Use the image file
  print('Image path: ${image.path}');
}
```

### Multiple Selection with Constraints

```dart
final List<XFile> images = await picker.pickMultiImage(
  maxWidth: 800,
  maxHeight: 600,
  imageQuality: 85,
  limit: 5,
);
```

### Camera with Preferences

```dart
final XFile? photo = await picker.pickImage(
  source: ImageSource.camera,
  preferredCameraDevice: CameraDevice.front,
  imageQuality: 90,
);
```

### Error Handling

```dart
try {
  final XFile? image = await picker.pickImage(source: ImageSource.camera);
  // Process image
} on PlatformException catch (e) {
  // Handle permission denied, camera unavailable, etc.
  print('Error: ${e.message}');
}
```

### Android Recovery Pattern

```dart
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  final ImagePicker picker = ImagePicker();
  
  @override
  void initState() {
    super.initState();
    _retrieveLostData();
  }
  
  Future<void> _retrieveLostData() async {
    final LostDataResponse response = await picker.retrieveLostData();
    if (response.file != null) {
      // Process recovered file
    }
  }
}
```

## Exported Types

The file exports several key types from the platform interface:

```dart 9:19:image_picker/lib/image_picker.dart
export 'package:image_picker_platform_interface/image_picker_platform_interface.dart'
  show
    CameraDevice,
    ImageSource,
    LostData,
    LostDataResponse,
    PickedFile,
    RetrieveType,
    XFile,
    kTypeImage,
    kTypeVideo;
```

These types provide the foundation for type-safe image and video handling across all supported platforms.
