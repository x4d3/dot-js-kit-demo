## Universal Camera Web Component

This is a platform independent web component that handles taking picture in browser both for face selfie and document photo scenario, both in landscape and in portrait, both for desktop with webcam and for mobile.

The component renders a video feed from the chosen camera and a button that allows user to take a picture. The component tries to open Full HD video stream. The Full HD 1080p and 720p image resolution is suitable for further processing in DOT products. Higher resolution is useless, smaller than 720p is insufficient.

If component succeeds to open the environment camera (back side of mobile), the video stream is not mirrored. In case of user camera, the video stream is mirrored for better user experience, but the returned photo is not mirrored.

If present, the video feed will be also rendered with an overlay mask.

Pressing the button captures the image from the media stream and returns it in the required format as blob.

### Known supported browsers

This component was tested with:

- Chrome on desktop (Windows and Linux)
- Firefox on desktop (Windows and Linux)
- Chrome on Android
- Firefox on Android
- Safari on iPhone\*

### Known issues

- the component does not work with Chrome (or other browsers) on iPhone as iOS does not provide camera access to other browsers than Safari

## Document Auto Capture Component

The goal of this component is to capture the best possible image with a document. This is achieved with document detection NN and image parameters analysis. Based on this data a user is instructed to move or turn in order to arrange suitable conditions for capturing an image.

### Integration

To integrate the library see the [docs](https://dot.pages.innovatrics.net/dot-documentation-public/technical/remote/dot-web-document/latest/documentation/).

### Auto Capture Process

1. Init camera using `cameraHandler.initWebcam()` on mount of `Camera` component and start loop with frames processing.
1. Process a camera preview frame by calling `documentDetector.detect()` (see the section below).
1. Set instruction for user returned by calling `documentDetector.detect()`.
1. Update state in `Camera` component:
   - If we got the second valid frame in a row, we enter the candidate selection phase for 1000 milliseconds and we consider the frame as an candidate (see the section below).
   - If  candidate selection phase finished, call `photoTakenCb()`. The process is over, return the captured face image. This happens outside the controller as the `Camera` component handles candidate selection phase timing. The best candidate can be retrieved by calling `documentDetector.getBestFrame();`.

#### Camera Preview Frame Processing

1. Crop two images for detection and parameters analysis.
   - _detection image_: placeholder area + padding. Call `getDetectionRect()` to get dimensions and `cropDocumentImage()` to get cropped image.
   - _"parameters analysis image_: placeholder area. Call `getPlaceholderRect()` to get dimensions and `cropDocumentImage()` to get cropped image.
1. Detect document corners in cropped _detection image_.
1. Analyze image parameters in cropped _parameters analysis image_. Currently requires sam.wasm statically hosted at the same relative path as the app, will support URL linking later. Imported by including `import sam_wasm_module from './sam/sam';` with sam.js in sam folder).
1. Evaluate detection and image parameters analysis data. In this step we have a list of validators. We validate the data with each validator and the result is a list of "hints". Each validator, which returns `false` produces a hint. Validation is handled by the `sam.getImageParameters()` function and `getValidationValues()` function. Thresholds are compared in `checkValidationValues()` function. List of validators:
   1. `DocumentNotDetectedValidator`
   1. `DocumentSmallValidator` (**Not implemented yet**)
   1. `DocumentLargeValidator` (**Not implemented yet**)
   1. `DocumentDoesNotFitPlaceholderValidator`
   1. `SharpnessLowValidator`
   1. `BrightnessLowValidator`
   1. `BrightnessHighValidator`
   1. `HotspotsScoreHighValidator`

#### Candidate Selection Phase

Time interval which starts in a moment when we evaluated second valid frame in a row. Duration is 1000 milliseconds (timing handled outside the controller in the `Camera` component). During this time interval we evaluate camera preview frames and calculate their score. The final image is the one with the highest score. It is guaranteed, that we have a candidate when the candidate selection phase finishes since we have assigned candidate already when the candidate selection phase starts (the last frame).

## Face Auto Capture Component

The goal of this component is to capture the best possible image with a face. This is achieved with face detection NN (https://github.com/tensorflow/tfjs-models/tree/master/blazeface). Based on this data a user is instructed to move or turn in order to arrange suitable conditions for capturing an image.

### Integration

To integrate the library see the [docs](https://dot.pages.innovatrics.net/dot-documentation-public/technical/remote/dot-web-face/latest/documentation/).

### Auto Capture Process

1. Init camera using `cameraHandler.initWebcam()` on mount of `Camera` component and start loop with frames processing.
1. Process a camera preview frame by calling `faceController.processImage()`. (see the section below).
1. Set instruction for user returned by calling `faceController.processImage()`.
1. Update state in `Camera` component:
   - If we got the second valid frame in a row, we enter the candidate selection phase for 1000 milliseconds and we consider the frame as an candidate (see the section below).
   - If  candidate selection phase finished, call `photoTakenCb()`. The process is over, return the captured face image. This happens outside the controller as the `Camera` component handles candidate selection phase timing. The best candidate can be retrieved by calling `faceController.getBestImage()`.

#### Camera Preview Frame Processing

1. Crop image: placeholder area + padding (square image required for best detection results).
1. Detect face in cropped image by calling `detectFace()` function.
1. Evaluate detection data by calling `validateDetectedFace()` function. In this step we have a list of validators. We validate the data with each validator and the result is a list of "hints". Each validator, which returns `false` produces a hint. List of validators:
   1. `FaceNotDetectedValidator`
   1. `FaceSmallValidator`
   1. `FaceLargeValidator`
   1. `FaceNotCenteredValidator`

#### Candidate Selection Phase

Time interval which starts in a moment when we evaluated second valid frame in a row. Duration is 1000 milliseconds (timing handled outside the controller in the Camera component). During this time interval we evaluate camera preview frames and calculate their score. The final image is the one with the highest score. It is guaranteed, that we have a candidate when the candidate selection phase finishes since we have assigned candidate already when the candidate selection phase starts (the last frame).
