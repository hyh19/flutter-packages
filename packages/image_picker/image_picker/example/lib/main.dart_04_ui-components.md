# Image Picker Example App - UI Components

## Main Scaffold Layout

The app uses a `Scaffold` with a clean, functional layout optimized for media picking operations:

```dart 333:494:image_picker/example/lib/main.dart
return Scaffold(
  appBar: AppBar(title: Text(widget.title!)),
  body: Center(
      child:
          !kIsWeb && defaultTargetPlatform == TargetPlatform.android
              ? FutureBuilder<void>(
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
              : _handlePreview(),
  ),
  floatingActionButton: Column(
    mainAxisAlignment: MainAxisAlignment.end,
    children: <Widget>[
      // FAB buttons...
    ],
  ),
);
```

## Floating Action Button Layout

The app features a vertical column of floating action buttons, each serving a specific media picking function:

### Image-Related Buttons

```dart 373:446:image_picker/example/lib/main.dart
Semantics(
  label: 'image_picker_example_from_gallery',
  child: FloatingActionButton(
    onPressed: () {
      isVideo = false;
      _onImageButtonPressed(ImageSource.gallery, context: context);
    },
    heroTag: 'image0',
    tooltip: 'Pick image from gallery',
    child: const Icon(Icons.photo),
  ),
),
Padding(
  padding: const EdgeInsets.only(top: 16.0),
  child: FloatingActionButton(
    onPressed: () {
      isVideo = false;
      _onImageButtonPressed(
        ImageSource.gallery,
        context: context,
        allowMultiple: true,
      );
    },
    heroTag: 'image1',
    tooltip: 'Pick multiple images',
    child: const Icon(Icons.photo_library),
  ),
),
Padding(
  padding: const EdgeInsets.only(top: 16.0),
  child: FloatingActionButton(
    onPressed: () {
      isVideo = false;
      _onImageButtonPressed(
        ImageSource.gallery,
        context: context,
        isMedia: true,
      );
    },
    heroTag: 'media',
    tooltip: 'Pick item from gallery',
    child: const Icon(Icons.photo_outlined),
  ),
),
Padding(
  padding: const EdgeInsets.only(top: 16.0),
  child: FloatingActionButton(
    onPressed: () {
      isVideo = false;
      _onImageButtonPressed(
        ImageSource.gallery,
        context: context,
        allowMultiple: true,
        isMedia: true,
      );
    },
    heroTag: 'multipleMedia',
    tooltip: 'Pick multiple items',
    child: const Icon(Icons.photo_library_outlined),
  ),
),
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

### Video-Related Buttons

```dart 447:491:image_picker/example/lib/main.dart
Padding(
  padding: const EdgeInsets.only(top: 16.0),
  child: FloatingActionButton(
    backgroundColor: Colors.red,
    onPressed: () {
      isVideo = true;
      _onImageButtonPressed(ImageSource.gallery, context: context);
    },
    heroTag: 'video',
    tooltip: 'Pick video from gallery',
    child: const Icon(Icons.video_file),
  ),
),
Padding(
  padding: const EdgeInsets.only(top: 16.0),
  child: FloatingActionButton(
    backgroundColor: Colors.red,
    onPressed: () {
      isVideo = true;
      _onImageButtonPressed(
        ImageSource.gallery,
        context: context,
        allowMultiple: true,
      );
    },
    heroTag: 'multiVideo',
    tooltip: 'Pick multiple videos',
    child: const Icon(Icons.video_library),
  ),
),
if (_picker.supportsImageSource(ImageSource.camera))
  Padding(
    padding: const EdgeInsets.only(top: 16.0),
    child: FloatingActionButton(
      backgroundColor: Colors.red,
      onPressed: () {
        isVideo = true;
        _onImageButtonPressed(ImageSource.camera, context: context);
      },
      heroTag: 'takeVideo',
      tooltip: 'Take a video',
      child: const Icon(Icons.videocam),
    ),
  ),
```

## Button Categories and Functions

### Single Selection Buttons

- **Gallery Image**: `Icons.photo` - Single image from gallery
- **Camera Photo**: `Icons.camera_alt` - Single photo from camera
- **Gallery Video**: `Icons.video_file` - Single video from gallery
- **Camera Video**: `Icons.videocam` - Single video from camera

### Multiple Selection Buttons

- **Multiple Images**: `Icons.photo_library` - Multiple images from gallery
- **Multiple Videos**: `Icons.video_library` - Multiple videos from gallery
- **Mixed Media**: `Icons.photo_outlined` - Single mixed media from gallery
- **Multiple Mixed Media**: `Icons.photo_library_outlined` - Multiple mixed media from gallery

## Configuration Dialog

For operations requiring user configuration (image compression, size limits, etc.), the app displays a modal dialog:

```dart 505:587:image_picker/example/lib/main.dart
Future<void> _displayPickImageDialog(
  BuildContext context,
  bool isMulti,
  OnPickImageCallback onPick,
) async {
  return showDialog(
    context: context,
    builder: (BuildContext context) {
      return AlertDialog(
        title: const Text('Add optional parameters'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            TextField(
              controller: maxWidthController,
              keyboardType: const TextInputType.numberWithOptions(
                decimal: true,
              ),
              decoration: const InputDecoration(
                hintText: 'Enter maxWidth if desired',
              ),
            ),
            TextField(
              controller: maxHeightController,
              keyboardType: const TextInputType.numberWithOptions(
                decimal: true,
              ),
              decoration: const InputDecoration(
                hintText: 'Enter maxHeight if desired',
              ),
            ),
            TextField(
              controller: qualityController,
              keyboardType: TextInputType.number,
              decoration: const InputDecoration(
                hintText: 'Enter quality if desired',
              ),
            ),
            if (isMulti)
              TextField(
                controller: limitController,
                keyboardType: TextInputType.number,
                decoration: const InputDecoration(
                  hintText: 'Enter limit if desired',
                ),
              ),
            ],
        ),
        actions: <Widget>[
          TextButton(
            child: const Text('CANCEL'),
            onPressed: () {
              Navigator.of(context).pop();
            },
          ),
          TextButton(
            child: const Text('PICK'),
            onPressed: () {
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
              onPick(width, height, quality, limit);
              Navigator.of(context).pop();
            },
          ),
        ],
      );
    },
  );
}
```

## Dialog Features

### Input Fields

1. **Max Width**: Decimal number input for maximum image width
2. **Max Height**: Decimal number input for maximum image height
3. **Quality**: Integer input for compression quality (0-100)
4. **Limit**: Integer input for maximum selection count (multi-selection only)

### Validation and Parsing

The dialog handles input validation:

- Empty strings are converted to `null`
- String values are parsed to appropriate numeric types
- Invalid inputs may cause runtime exceptions (basic error handling)

### Callback Pattern

The dialog uses a callback pattern (`OnPickImageCallback`) to pass configuration back to the calling method:

```dart 590:596:image_picker/example/lib/main.dart
typedef OnPickImageCallback =
  void Function(
    double? maxWidth,
    double? maxHeight,
    int? quality,
    int? limit,
  );
```

## Accessibility Features

### Semantic Labels

Each major UI element includes semantic labels for screen readers:

```dart
Semantics(
  label: 'image_picker_example_from_gallery',
  child: FloatingActionButton(...),
)
```

### Tooltips

All buttons include descriptive tooltips:

- "Pick image from gallery"
- "Pick multiple images"
- "Take a photo"
- "Pick video from gallery"
- "Take a video"

## Visual Design

### Color Coding

- **Default buttons**: Standard Material Design colors
- **Video buttons**: Red background (`Colors.red`) to distinguish from image buttons

### Spacing

- **Button spacing**: 16.0 pixels between buttons
- **Icon consistency**: Related functions use similar icon styles

### Layout Strategy

- **Vertical stacking**: Buttons arranged in a column from top to bottom
- **Right alignment**: Positioned on the right side of the screen
- **Bottom positioning**: Aligned to the bottom of the screen

This UI design provides an intuitive, accessible interface for testing various image picker functionalities with clear visual distinctions between different operation types.
