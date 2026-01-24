# Image Picker Example App - Error Handling and Edge Cases

## Error Handling Architecture

The app implements comprehensive error handling across different layers of the media picking and display system.

## Primary Error States

### Picking Operation Errors

All media picking operations are wrapped in try-catch blocks:

```dart 133:137:image_picker/example/lib/main.dart
} catch (e) {
  setState(() {
    _pickImageError = e;
  });
}
```

The pattern is consistent across:

- Single image picking (`pickImage`)
- Multiple image picking (`pickMultiImage`)
- Single media picking (`pickMedia`)
- Multiple media picking (`pickMultipleMedia`)

### Error State Management

```dart 48:48:image_picker/example/lib/main.dart
dynamic _pickImageError;
```

Errors are stored as dynamic type to accommodate different exception types from the image_picker package.

## Display Error Handling

### Error Widget Generation

```dart 496:503:image_picker/example/lib/main.dart
Text? _getRetrieveErrorWidget() {
  if (_retrieveDataError != null) {
    final Text result = Text(_retrieveDataError!);
    _retrieveDataError = null;
    return result;
  }
  return null;
}
```

The "consume-on-display" pattern ensures error messages are shown once and then cleared.

### Error Display in UI

Errors are displayed in preview areas:

```dart 274:279:image_picker/example/lib/main.dart
} else if (_pickImageError != null) {
  return Text(
    'Pick image error: $_pickImageError',
    textAlign: TextAlign.center,
  );
}
```

## Android-Specific Error Handling

### Lost Data Recovery

Android has unique behavior where the system may kill the app during media selection:

```dart 307:329:image_picker/example/lib/main.dart
Future<void> retrieveLostData() async {
  final LostDataResponse response = await _picker.retrieveLostData();
  if (response.isEmpty) {
    return;
  }
  if (response.file != null) {
    if (response.type == RetrieveType.video) {
      isVideo = true;
      await _playVideo(response.file);
    } else {
      isVideo = false;
      setState(() {
        if (response.files == null) {
          _setImageFileListFromFile(response.file);
        } else {
          _mediaFileList = response.files;
        }
      });
    }
  } else {
    _retrieveDataError = response.exception!.code;
  }
}
```

This method handles:

- **App Termination**: When Android kills the app during picking
- **Data Recovery**: Retrieves previously selected files
- **Type Detection**: Determines if recovered data is video or image
- **State Restoration**: Properly updates app state with recovered data

### FutureBuilder Integration

```dart 338:365:image_picker/example/lib/main.dart
FutureBuilder<void>(
  future: retrieveLostData(),
  builder: (
    BuildContext context,
    AsyncSnapshot<void> snapshot,
  ) {
    switch (snapshot.connectionState) {
      case ConnectionState.none:
      case ConnectionState.waiting:
        return const Text(
          'You have not yet picked an image.',
          textAlign: TextAlign.center,
        );
      case ConnectionState.done:
        return _handlePreview();
      case ConnectionState.active:
        if (snapshot.hasError) {
          return Text(
            'Pick image/video error: ${snapshot.error}}',
            textAlign: TextAlign.center,
          );
        } else {
          return const Text(
            'You have not yet picked an image.',
            textAlign: TextAlign.center,
          );
        }
    }
  },
)
```

## Platform-Specific Edge Cases

### Web Platform Limitations

#### Autoplay Restrictions

```dart 72:77:image_picker/example/lib/main.dart
// In web, most browsers won't honor a programmatic call to .play
// if the video has a sound track (and is not muted).
// Mute the video so it auto-plays in web!
// This is not needed if the call to .play is the result of user
// interaction (clicking on a "play" button, for example).
const double volume = kIsWeb ? 0.0 : 1.0;
```

Web browsers restrict autoplay with sound, requiring videos to be muted for programmatic playback.

#### File Access Differences

```dart 250:268:image_picker/example/lib/main.dart
kIsWeb
    ? Image.network(_mediaFileList![index].path)
    : (mime == null || mime.startsWith('image/')
        ? Image.file(
          File(_mediaFileList![index].path),
          errorBuilder: (
            BuildContext context,
            Object error,
            StackTrace? stackTrace,
          ) {
            return const Center(
              child: Text('This image type is not supported'),
            );
          },
        )
        : _buildInlineVideoPlayer(index)),
```

Web platform uses network URLs while native platforms use file system access.

### Camera Availability

```dart 434:446:image_picker/example/lib/main.dart
if (_picker.supportsImageSource(ImageSource.camera))
  Padding(
    padding: const EdgeInsets.only(top: 16.0),
    child: FloatingActionButton(
      onPressed: () {
        isVideo = false;
        _onImageButtonPressed(ImageSource.camera, context: context);
      },
      heroTag: 'image2',
      tooltip: 'Take a photo',
      child: const Icon(Icons.camera_alt),
    ),
  ),
```

Camera buttons are conditionally displayed based on device capabilities.

## Media Format Edge Cases

### Unsupported Image Formats

```dart 258:267:image_picker/example/lib/main.dart
errorBuilder: (
  BuildContext context,
  Object error,
  StackTrace? stackTrace,
) {
  return const Center(
    child: Text('This image type is not supported'),
  );
},
```

Provides fallback UI when Flutter cannot decode certain image formats.

### MIME Type Detection

```dart 246:246:image_picker/example/lib/main.dart
final String? mime = lookupMimeType(_mediaFileList![index].path);
```

Uses MIME type detection to differentiate between images and videos in mixed media lists.

## Memory and Resource Management

### Video Controller Disposal

```dart 210:216:image_picker/example/lib/main.dart
Future<void> _disposeVideoController() async {
  if (_toBeDisposed != null) {
    await _toBeDisposed!.dispose();
  }
  _toBeDisposed = _controller;
  _controller = null;
}
```

Implements careful disposal to prevent memory leaks:

- **Deferred Disposal**: Controllers are marked for disposal but cleaned up asynchronously
- **Null Safety**: Prevents crashes from accessing disposed controllers

### Widget Lifecycle Management

```dart 192:199:image_picker/example/lib/main.dart
@Override
void deactivate() {
  if (_controller != null) {
    _controller!.setVolume(0.0);
    _controller!.pause();
  }
  super.deactivate();
}
```

Ensures video playback stops when the widget becomes inactive.

## Input Validation Edge Cases

### Configuration Dialog Validation

```dart 563:581:image_picker/example/lib/main.dart
final double? width =
    maxWidthController.text.isNotEmpty
        ? double.parse(maxWidthController.text)
        : null;
final double? height =
    maxHeightController.text.isNotEmpty
        ? double.parse(maxHeightController.text)
        : null;
final int? quality =
    qualityController.text.isNotEmpty
        ? int.parse(qualityController.text)
        : null;
final int? limit =
    limitController.text.isNotEmpty
        ? int.parse(limitController.text)
        : null;
```

Basic validation handles:

- **Empty Strings**: Converted to null values
- **Type Conversion**: String to numeric types
- **Invalid Input**: May throw exceptions (basic error handling)

## Network and Permission Errors

### Permission Denied Scenarios

The app doesn't explicitly handle permission denials but relies on the image_picker package to surface these as exceptions that get caught by the general error handling.

### Network Timeouts (Web)

For web platform network operations, timeouts and connectivity issues would be handled by the underlying network libraries.

## State Consistency Edge Cases

### Mounted State Checks

```dart 62:62:image_picker/example/lib/main.dart
if (file != null && mounted) {
```

Ensures operations don't proceed if the widget has been disposed.

### Context Availability

```dart 94:94:image_picker/example/lib/main.dart
if (context.mounted) {
```

Checks for context availability before showing dialogs or performing navigation.

## Performance Edge Cases

### Large File Handling

The app doesn't implement specific handling for very large files but relies on the image_picker package's built-in limitations and the device's available memory.

### Multiple Video Playback

When displaying multiple videos in a list, each gets its own controller:

```dart 287:297:image_picker/example/lib/main.dart
Widget _buildInlineVideoPlayer(int index) {
  final VideoPlayerController controller = VideoPlayerController.file(
    File(_mediaFileList![index].path),
  );
  const double volume = kIsWeb ? 0.0 : 1.0;
  controller.setVolume(volume);
  controller.initialize();
  controller.setLooping(true);
  controller.play();
  return Center(child: AspectRatioVideo(controller));
}
```

This can be resource-intensive with many videos.

## Recovery Mechanisms

### Error State Reset

The app implements various recovery mechanisms:

- **Error Message Clearing**: Errors are cleared after display
- **State Reset**: Failed operations don't leave the app in inconsistent states
- **Resource Cleanup**: Proper disposal ensures clean recovery

This comprehensive error handling system ensures the app remains stable and provides meaningful feedback to users across various failure scenarios and platform-specific edge cases.
