# Image Picker Example App - Image/Video Picking Functionality

## Core Picking Method

The main image/video picking functionality is implemented in the `_onImageButtonPressed` method, which handles all media selection operations with different configurations.

```dart 85:190:image_picker/example/lib/main.dart
Future<void> _onImageButtonPressed(
  ImageSource source, {
  required BuildContext context,
  bool allowMultiple = false,
  bool isMedia = false,
}) async {
  if (_controller != null) {
    await _controller!.setVolume(0.0);
  }
  if (context.mounted) {
    if (isVideo) {
      final List<XFile> files;
      if (allowMultiple) {
        files = await _picker.pickMultiVideo();
      } else {
        final XFile? file = await _picker.pickVideo(
          source: source,
          maxDuration: const Duration(seconds: 10),
        );
        files = <XFile>[if (file != null) file];
      }
      // Just play the first file, to keep the example simple.
      await _playVideo(files.firstOrNull);
    } else if (allowMultiple) {
      await _displayPickImageDialog(context, true, (
        double? maxWidth,
        double? maxHeight,
        int? quality,
        int? limit,
      ) async {
        try {
          final List<XFile> pickedFileList =
              isMedia
                  ? await _picker.pickMultipleMedia(
                    maxWidth: maxWidth,
                    maxHeight: maxHeight,
                    imageQuality: quality,
                    limit: limit,
                  )
                  : await _picker.pickMultiImage(
                    maxWidth: maxWidth,
                    maxHeight: maxHeight,
                    imageQuality: quality,
                    limit: limit,
                  );
          setState(() {
            _mediaFileList = pickedFileList;
          });
        } catch (e) {
          setState(() {
            _pickImageError = e;
          });
        }
      });
    } else if (isMedia) {
      await _displayPickImageDialog(context, false, (
        double? maxWidth,
        double? maxHeight,
        int? quality,
        int? limit,
      ) async {
        try {
          final List<XFile> pickedFileList = <XFile>[];
          final XFile? media = await _picker.pickMedia(
            maxWidth: maxWidth,
            maxHeight: maxHeight,
            imageQuality: quality,
          );
          if (media != null) {
            pickedFileList.add(media);
            setState(() {
              _mediaFileList = pickedFileList;
            });
          }
        } catch (e) {
          setState(() {
            _pickImageError = e;
          });
        }
      });
    } else {
      await _displayPickImageDialog(context, false, (
        double? maxWidth,
        double? maxHeight,
        int? quality,
        int? limit,
      ) async {
        try {
          final XFile? pickedFile = await _picker.pickImage(
            source: source,
            maxWidth: maxWidth,
            maxHeight: maxHeight,
            imageQuality: quality,
          );
          setState(() {
            _setImageFileListFromFile(pickedFile);
          });
        } catch (e) {
          setState(() {
            _pickImageError = e;
          });
        }
      });
    }
  }
}
```

## Picking Methods Overview

The app supports various picking methods through the `ImagePicker` class:

### 1. Single Image Picking (`pickImage`)

```dart
final XFile? pickedFile = await _picker.pickImage(
  source: source,
  maxWidth: maxWidth,
  maxHeight: maxHeight,
  imageQuality: quality,
);
```

### 2. Multiple Image Picking (`pickMultiImage`)

```dart
final List<XFile> pickedFileList = await _picker.pickMultiImage(
  maxWidth: maxWidth,
  maxHeight: maxHeight,
  imageQuality: quality,
  limit: limit,
);
```

### 3. Single Video Picking (`pickVideo`)

```dart
final XFile? file = await _picker.pickVideo(
  source: source,
  maxDuration: const Duration(seconds: 10),
);
```

### 4. Multiple Video Picking (`pickMultiVideo`)

```dart
final List<XFile> files = await _picker.pickMultiVideo();
```

### 5. Mixed Media Picking (`pickMedia` and `pickMultipleMedia`)

```dart
final XFile? media = await _picker.pickMedia(
  maxWidth: maxWidth,
  maxHeight: maxHeight,
  imageQuality: quality,
);

final List<XFile> pickedFileList = await _picker.pickMultipleMedia(
  maxWidth: maxWidth,
  maxHeight: maxHeight,
  imageQuality: quality,
  limit: limit,
);
```

## Configuration Options

### Image Configuration Parameters

- **`maxWidth`**: Maximum width in pixels for image resizing
- **`maxHeight`**: Maximum height in pixels for image resizing
- **`imageQuality`**: Compression quality (0-100), where 100 is highest quality

### Video Configuration Parameters

- **`maxDuration`**: Maximum recording duration for video capture
- **`source`**: ImageSource.camera or ImageSource.gallery

### Selection Limits

- **`limit`**: Maximum number of items to select in multi-selection modes

## Source Types

The app supports two main image sources:

### Camera Source (`ImageSource.camera`)

- Captures new photos/videos using device camera
- Only available on mobile devices with camera support
- Respects camera permissions

### Gallery Source (`ImageSource.gallery`)

- Selects existing photos/videos from device storage
- Available on all platforms
- Respects storage permissions

## Picking Logic Flow

### Video Mode (`isVideo = true`)

1. **Single Video**: Uses `pickVideo()` with duration limit
2. **Multiple Videos**: Uses `pickMultiVideo()` without configuration options
3. **Automatic Playback**: First selected video is automatically played

### Image Mode (`isVideo = false`)

1. **Single Image**: Uses `pickImage()` with full configuration options
2. **Multiple Images**: Uses `pickMultiImage()` with configuration and limits
3. **Mixed Media**: Uses `pickMedia()` or `pickMultipleMedia()` for combined content

### Media Mode (`isMedia = true`)

Similar to image mode but allows selection of both images and videos from gallery.

## Error Handling

All picking operations are wrapped in try-catch blocks:

```dart
try {
  // Picking operation
  setState(() {
    _mediaFileList = pickedFileList;
  });
} catch (e) {
  setState(() {
    _pickImageError = e;
  });
}
```

Errors are stored in `_pickImageError` and displayed in the UI.

## Platform Considerations

### Camera Support Check

```dart
if (_picker.supportsImageSource(ImageSource.camera))
```

The app checks for camera availability before showing camera-related buttons, ensuring compatibility across different device types.

### Web Platform Behavior

For web platforms, video playback is muted by default due to browser autoplay policies:

```dart
const double volume = kIsWeb ? 0.0 : 1.0;
```

## Video Playback Integration

When videos are selected, they're automatically played using the `_playVideo` method:

```dart
await _playVideo(files.firstOrNull);
```

This provides immediate feedback to users about their video selection.

## Configuration Dialog

Complex picking operations (with configuration options) use a dialog interface to collect user preferences before executing the pick operation. This is handled by the `_displayPickImageDialog` method, which will be covered in detail in the UI components document.
