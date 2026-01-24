# Image Picker Example App - Video Playback and Display Features

## Video Playback System

The app implements comprehensive video playback functionality using the `video_player` package, with special handling for different platforms and cross-platform compatibility.

## Core Video Playback Method

The `_playVideo` method handles video initialization and playback:

```dart 61:83:image_picker/example/lib/main.dart
Future<void> _playVideo(XFile? file) async {
  if (file != null && mounted) {
    await _disposeVideoController();
    final VideoPlayerController controller;
    if (kIsWeb) {
      controller = VideoPlayerController.networkUrl(Uri.parse(file.path));
    } else {
      controller = VideoPlayerController.file(File(file.path));
    }
    _controller = controller;
    // In web, most browsers won't honor a programmatic call to .play
    // if the video has a sound track (and is not muted).
    // Mute the video so it auto-plays in web!
    // This is not needed if the call to .play is the result of user
    // interaction (clicking on a "play" button, for example).
    const double volume = kIsWeb ? 0.0 : 1.0;
    await controller.setVolume(volume);
    await controller.initialize();
    await controller.setLooping(true);
    await controller.play();
    setState(() {});
  }
}
```

## Platform-Specific Video Handling

### Web Platform Considerations

For web platforms, the app implements special handling due to browser autoplay policies:

- **Muted Playback**: Videos are muted (`volume = 0.0`) to enable autoplay
- **Network URL**: Uses `VideoPlayerController.networkUrl()` instead of file controller
- **User Interaction**: Comments indicate this is for programmatic playback

### Mobile/Desktop Platforms

For native platforms:

- **File Controller**: Uses `VideoPlayerController.file()` for local files
- **Normal Volume**: Sets volume to 1.0 for audible playback
- **Direct File Access**: Leverages platform file system access

## Custom Video Player Widget

The `AspectRatioVideo` widget provides a specialized video player component:

```dart 598:646:image_picker/example/lib/main.dart
class AspectRatioVideo extends StatefulWidget {
  const AspectRatioVideo(this.controller, {super.key});

  final VideoPlayerController? controller;

  @override
  AspectRatioVideoState createState() => AspectRatioVideoState();
}

class AspectRatioVideoState extends State<AspectRatioVideo> {
  VideoPlayerController? get controller => widget.controller;
  bool initialized = false;

  void _onVideoControllerUpdate() {
    if (!mounted) {
      return;
    }
    if (initialized != controller!.value.isInitialized) {
      initialized = controller!.value.isInitialized;
      setState(() {});
    }
  }

  @override
  void initState() {
    super.initState();
    controller!.addListener(_onVideoControllerUpdate);
  }

  @override
  void dispose() {
    controller!.removeListener(_onVideoControllerUpdate);
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    if (initialized) {
      return Center(
        child: AspectRatio(
          aspectRatio: controller!.value.aspectRatio,
          child: VideoPlayer(controller!),
        ),
      );
    } else {
      return Container();
    }
  }
}
```

## Video Widget Features

### Aspect Ratio Maintenance

The widget ensures proper video display by:

- **Dynamic Aspect Ratio**: Uses `controller.value.aspectRatio` for correct proportions
- **Centered Display**: Centers the video in available space
- **Responsive Layout**: Adapts to different screen sizes

### Initialization State Management

The widget tracks video initialization state:

- **State Variable**: `initialized` flag tracks readiness
- **Listener Pattern**: Responds to controller state changes
- **Conditional Rendering**: Shows video only when ready

### Resource Management

Proper cleanup is implemented:

- **Listener Removal**: Removes update listeners on dispose
- **State Safety**: Checks `mounted` status before updates

## Preview System

The app uses a unified preview system that switches between image and video display:

```dart 299:305:image_picker/example/lib/main.dart
Widget _handlePreview() {
  if (isVideo) {
    return _previewVideo();
  } else {
    return _previewImages();
  }
}
```

## Video Preview Implementation

```dart 218:233:image_picker/example/lib/main.dart
Widget _previewVideo() {
  final Text? retrieveError = _getRetrieveErrorWidget();
  if (retrieveError != null) {
    return retrieveError;
  }
  if (_controller == null) {
    return const Text(
      'You have not yet picked a video',
      textAlign: TextAlign.center,
    );
  }
  return Padding(
    padding: const EdgeInsets.all(10.0),
    child: AspectRatioVideo(_controller),
  );
}
```

### Video Preview States

1. **Error State**: Shows error message if retrieval failed
2. **Empty State**: Displays message when no video is selected
3. **Active State**: Shows video player with proper padding

## Image Preview Implementation

```dart 235:285:image_picker/example/lib/main.dart
Widget _previewImages() {
  final Text? retrieveError = _getRetrieveErrorWidget();
  if (retrieveError != null) {
    return retrieveError;
  }
  if (_mediaFileList != null) {
    return Semantics(
      label: 'image_picker_example_picked_images',
      child: ListView.builder(
        key: UniqueKey(),
        itemBuilder: (BuildContext context, int index) {
          final String? mime = lookupMimeType(_mediaFileList![index].path);

          // Why network for web?
          // See https://pub.dev/packages/image_picker_for_web#limitations-on-the-web-platform
          return Semantics(
            label: 'image_picker_example_picked_image',
            child:
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
          );
        },
        itemCount: _mediaFileList!.length,
      ),
    );
  } else if (_pickImageError != null) {
    return Text(
      'Pick image error: $_pickImageError',
      textAlign: TextAlign.center,
    );
  } else {
    return const Text(
      'You have not yet picked an image.',
      textAlign: TextAlign.center,
    );
  }
}
```

## Mixed Media Handling

The image preview system handles mixed media content:

### MIME Type Detection

```dart
final String? mime = lookupMimeType(_mediaFileList![index].path);
```

Uses the `mime` package to determine file types.

### Conditional Rendering

- **Images**: Standard `Image.file` or `Image.network` widgets
- **Videos in Image List**: Inline video player for video files in image mode
- **Platform-Specific Display**: Web uses network URLs, native uses file paths

## Inline Video Player for Mixed Lists

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

This creates individual video controllers for each video in a mixed media list, ensuring each video can play independently.

## Error Handling in Display

### Image Loading Errors

```dart
Image.file(
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
),
```

Provides fallback UI for unsupported image formats.

### Video Playback Errors

Video errors are handled through the controller's error states and the general error display system.

## Performance Considerations

### Controller Reuse

The app reuses video controllers when possible to minimize resource allocation.

### List View Optimization

Uses `UniqueKey()` for the ListView to ensure proper rebuilding when content changes.

### Memory Management

Proper disposal of controllers prevents memory leaks, especially important for video content.

This comprehensive video playback and display system ensures smooth media preview across different platforms and content types, with robust error handling and performance optimization.
