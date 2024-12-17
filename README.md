In this step by step tutorial, we will learn how to use Face-api.js and vanilla JavaScript to detect faces and expressions in real-time, all that from the browser, no back-end needed.

### **What is Face-api.js?**

[Face-api.js](https://justadudewhohacks.github.io/face-api.js/) is a JavaScript API for face detection and face recognition directly in the browser implemented build on top of the tensorflow.js

### **What are we building?**

In this tutorial we will build a simple application that streams a video from the webcam of the PC or the camera of the phone, search for any faces present in it, and show a blue rectangle containing: the age, the gender and the expression for each face.

### **List of models that we will use in our Tutorial**

#### **Tiny Face Detector**

Very performant, real time face detector, This model is extremely mobile and web friendly, thus it should be your GO-TO face detector on mobile devices and resource limited clients. The size of the quantized model is only 190 KB (**tiny\_face\_detector\_model**).

#### **68 Point Face Landmark Detection Models**

This package implements a very lightweight and fast, yet accurate 68 point face landmark detector. The default model has a size of only 350kb (**face\_landmark\_68\_model**) and the tiny model is only 80kb (**face\_landmark\_68\_tiny\_model**). Both models employ the ideas of depth wise separable convolutions as well as densely connected blocks. The models have been trained on a dataset of ~35k face images labelled with 68 face landmark points.

#### **Age Gender model**

Age estimation and gender recognition model from detected faces ( for each face detected in the image or the video).

#### **Face expression model**

Face expression recognition model, can detect expressions in an image or a video of each face. 

### **Let’s code**

Our application structure will look like this:

```plaintext
/Root
 ├─ /models
 ├─ app.js
 ├─ face-api.min.js
 ├─ styles.css
 └─ index.html
```

#### **./models**

Contain the files of all the trained models to use with our Face-api.js API.

%[https://github.com/adelpro/face-api-detection/tree/0796c1df82306cd8b6e7e837036399559572b23c/models] 

#### **face-api.min.js**

Contain the code of our face-api API (minified), you can download this file from here:

%[https://github.com/adelpro/face-api-detection/blob/0796c1df82306cd8b6e7e837036399559572b23c/face-api.min.js] 

#### **styles.css**

A simple CSS files to style our application:

```css
.container {

 display: flex;

 justify-content: centre;

 flex-direction: column;

 align-items: centre;

 height: 100vh;

 text-align: centre;

 flex: 1;

}


.video-container {

 position: relative;

}


canvas {

 position: absolute;

 top: 0;

 left: 0;

}
```

As you can see we have a canvas element that will be positioned over our video, and we will use it to draw our face detection.

#### **index.html**

This file contain all the necessary HTML code for your application, we will do all the logic in our app.js

```xml
<!DOCTYPE html>

<html lang="en">

<head>

 <meta charset="UTF-8">

 <meta http-equiv="X-UA-Compatible" content="IE=edge">

 <meta name="viewport" content="width=device-width, initial-scale=1.0">

 <title>Face-API AI Real-Time Facial Detection</title>

 <link rel="stylesheet" type="text/css" href="styles.css">

 <script defer src="face-api.min.js"></script>

 <script defer src="app.js"></script>

</head>


<body>

 <main class="container">

   <h1>Real-Time facial detection</h1>

   <p>We will use Javascript and Face-API to create a real time facial detection system</p>

   <div id="video-container" class="video-container">

     <video id="video" autoplay muted width="500" height="500"></video>

   </div>

 </main>

</body>

</html>
```

#### **app.js**

```javascript
const video = document.getElementById("video");

const videoContainer = document.getElementById("video-container");

const MODEL_URI = "/models";

Promise.all([

 faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URI),

 faceapi.nets.faceLandmark68Net.loadFromUri(MODEL_URI),

 faceapi.nets.faceRecognitionNet.loadFromUri(MODEL_URI),

 faceapi.nets.faceExpressionNet.loadFromUri(MODEL_URI),

 faceapi.nets.ageGenderNet.loadFromUri(MODEL_URI),

])

 .then(playVideo)

 .catch((err) => {

   console.log(err);

 });
function playVideo() {

 if (!navigator.mediaDevices) {

   console.error("mediaDevices not supported");

   return;

 }

 navigator.mediaDevices

   .getUserMedia({

     video: {

       width: { min: 640, ideal: 1280, max: 1920 },

       height: { min: 360, ideal: 720, max: 1080 },

     },

     audio: false,

   })

   .then(function (stream) {

     video.srcObject = stream;

   })

   .catch(function (err) {

     console.log(err);

   });

}

video.addEventListener("play", () => {

 // Creating the canvas

 const canvas = faceapi.createCanvasFromMedia(video);


 // This will force the use of a software (instead of hardware accelerated)

 // Enable only for low configurations

 canvas.willReadFrequently = true;

 videoContainer.appendChild(canvas);


 // Resizing the canvas to cover the video element

 const canvasSize = { width: video.width, height: video.height };

 faceapi.matchDimensions(canvas, canvasSize);


 setInterval(async () => {

   const detections = await faceapi

     .detectAllFaces(video, new faceapi.TinyFaceDetectorOptions())

     .withFaceLandmarks()

     .withFaceExpressions()

     .withAgeAndGender();


   // Set detections size to the canvas size

   const DetectionsArray = faceapi.resizeResults(detections, canvasSize);

   canvas.getContext("2d").clearRect(0, 0, canvas.width, canvas.height);

   detectionsDraw(canvas, DetectionsArray);

 }, 10);

});


// Drawing our detections above the video

function detectionsDraw(canvas, DetectionsArray) {

 // Adjust the size of the detection canvas

 faceapi.draw.drawDetections(canvas, DetectionsArray);

 faceapi.draw.drawFaceLandmarks(canvas, DetectionsArray);

 faceapi.draw.drawFaceExpressions(canvas, DetectionsArray);


 // Drawing AGE and GENDER

 DetectionsArray.forEach((detection) => {

   const box = detection.detection.box;

   const drawBox = new faceapi.draw.DrawBox(box, {

     label: ${Math.round(detection.age)}y, ${detection.gender},

   });

   drawBox.draw(canvas);

 });

}

Now we will go throw app.js, and explain different code blocks.

const MODEL_URI = "/models";

Promise.all([

 faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URI),

 faceapi.nets.faceLandmark68Net.loadFromUri(MODEL_URI),

 faceapi.nets.faceRecognitionNet.loadFromUri(MODEL_URI),

 faceapi.nets.faceExpressionNet.loadFromUri(MODEL_URI),

 faceapi.nets.ageGenderNet.loadFromUri(MODEL_URI),

])

 .then(playVideo)

 .catch((err) => {

   console.log(err);

 });
```

In this code block, we are loading our models files from the ‘/models’ folder using **Promise.all\[\]**, which allows us to wait for all models to load (it will take some time, depending on your hardware). 

Then, we call the function **playVideo** and of course catch any errors and show them in the console.

Our **playVideo()** function

First we check if the browser support [mediaDevices API](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices) , if not we stop and execution of the function and show an error in the console: “mediaDevices not supported”

Then we call the mediaDevice.getUserMedia: this will show a notification to the user, asking permission to access the camera ( no sound in our case),if the user refuses, we throw an error.

If all goes well, we assign the stream to our video element:

video.srcObject = stream;

Now our video element starts showing a live stream from the camera.

video.addEventListener("play", () =&gt; {

 // Creating the canvas

 const canvas = faceapi.createCanvasFromMedia(video);

```javascript
function playVideo() {

 if (!navigator.mediaDevices) {

   console.error("mediaDevices not supported");

   return;

 }

 navigator.mediaDevices

   .getUserMedia({

     video: {

       width: { min: 640, ideal: 1280, max: 1920 },

       height: { min: 360, ideal: 720, max: 1080 },

     },

     audio: false,

   })

   .then(function (stream) {

     video.srcObject = stream;

   })

   .catch(function (err) {

     console.log(err);

   });

}
 // This will force the use of a software (instead of hardware accelerated)

 // Enable only for low configurations

 canvas.willReadFrequently = true;

 videoContainer.appendChild(canvas);


 // Resizing the canvas to cover the video element

 const canvasSize = { width: video.width, height: video.height };

 faceapi.matchDimensions(canvas, canvasSize);


 setInterval(async () => {

   const detections = await faceapi

     .detectAllFaces(video, new faceapi.TinyFaceDetectorOptions())

     .withFaceLandmarks()

     .withFaceExpressions()

     .withAgeAndGender();


   // Set detections size to the canvas size

   const DetectionsArray = faceapi.resizeResults(detections, canvasSize);

   canvas.getContext("2d").clearRect(0, 0, canvas.width, canvas.height);

   detectionsDraw(canvas, DetectionsArray);

 }, 10);

});
```

Now come's this block of code, this will be executed when the video starts playing.

In this block of code we will start drawing our different detection: face, landmarks, expressions, age, gender. All that using the canvas element.

First: creating the canvas and assign it to the video element

```javascript
const canvas = faceapi.createCanvasFromMedia(video);

// This will force the use of a software (instead of hardware accelerated)

 // Enable only for low configurations

 canvas.willReadFrequently = true;

 videoContainer.appendChild(canvas);
```

Then resize the canvas to muth the video element

```javascript
// Resizing the canvas to cover the video element

 const canvasSize = { width: video.width, height: video.height };

 faceapi.matchDimensions(canvas, canvasSize);

Start a timer that will search for face detection in the video stream, and draw the results over our canvas

setInterval(async () => {

   const detections = await faceapi

     .detectAllFaces(video, new faceapi.TinyFaceDetectorOptions())

     .withFaceLandmarks()

     .withFaceExpressions()

     .withAgeAndGender();


   // Set detections size to the canvas size

   const DetectionsArray = faceapi.resizeResults(detections, canvasSize);

   canvas.getContext("2d").clearRect(0, 0, canvas.width, canvas.height);

   detectionsDraw(canvas, DetectionsArray);

 }, 10);
```

### **Full complete source code**

%[https://github.com/adelpro/face-api-detection]
