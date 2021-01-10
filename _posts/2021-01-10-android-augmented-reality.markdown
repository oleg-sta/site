---
layout: post
title:  "Android Augmented reality effect implementation"
date:   2021-01-10 11:25:01
categories: android
---
# Android Augmented reality effect implementation

In this article I'll give you understanding how to write application on Android with effects on the face from camera in realtime.
We will go through all the key points here.

![Demo of result]({{site.url}}/assets/face-mask-demo.gif)

## Working with camera

The first thing we need to do is to get image from camera.
Android API provides us package `android.hardware.camera2`.
First you need to choose camera, by getting all the available cameras `Camera.getNumberOfCameras()`.
Here you should specify which camera you want to use, and then open camera:
```
    mCamera = Camera.open(cameraIndex);
```
Then you have to choose preview parameters:
```
    Camera.Parameters params = mCamera.getParameters();
    params.setPreviewFormat(ImageFormat.NV21);
    List<Camera.Size> psize = params.getSupportedPreviewSizes();
```
Among `List` of preview size you have to choose the most appropriate.
It's better to choose with nearly size of the screen.

The other complex thing is to start preview. You need to establish buffer and preview callback
```
    int size = cameraWidth * cameraHeight;
    size  = size * ImageFormat.getBitsPerPixel(params.getPreviewFormat()) / 8;
    mBuffer = new byte[size];
    mCamera.addCallbackBuffer(mBuffer);
    mCamera.setPreviewCallbackWithBuffer(preview);
``` 
For preview you need to implements only one method `void onPreviewFrame(byte[] data, Camera camera)`.
Where `data` is preview frame from camera, you'll receive it in loop whenever preview calculated from Camera and you released method.
One thing to mention, our preview frame in `NV21` format, in short this format of picture.
Where it split on 2 parts: grey image and colour.
Which is very useful, because the next two steps is finding face and point, which is done on grey image and we don't need to make additional computation to find grey image, just get first part of the buffer.  

## Finding face

There are different techniques to find face on the screen.
We are going to use the most popular [haar features](https://en.wikipedia.org/wiki/Haar-like_feature).
This algorithm is already implements in [OpenCV](https://opencv.org/), it's one of the most popular library for computer vision.
It is written on `C/C++`. As machine learning, we need to train our model.
For this we need to have thousands photos with faces and without faces.
Then algorithm will be trained and will produce the model that could be used.
We don't need to do it, everything is already done with `face_detection` method along with the model. 

## Finding points on the face

On the face we should find specific points to understand face orientation and facial expression.
For this we are going to use [dlib](http://dlib.net/) library.
It's also popular library for machine learning written on C++.
Example of finding points on the face here [http://dlib.net/face_landmark_detection_ex.cpp.html](http://dlib.net/face_landmark_detection_ex.cpp.html).
As it's written you could use ready model [http://dlib.net/files/shape_predictor_68_face_landmarks.dat.bz2](http://dlib.net/files/shape_predictor_68_face_landmarks.dat.bz2).

## Orientation and facial expression

When we have 68 points on the screen we need to calculate face orientation and facial expression, e.g. opened jaw.
For this we are going to use 3d model for the face.
We need to have points on 3d face and corresponding points among 68 points.
If we want to find facial expression, we should solve equation:
```
dst = argmin X ∥ src1⋅ X − src2∥
```
by [SVD](https://en.wikipedia.org/wiki/Singular_value_decomposition) decomposition. Where:
* scr1 - 3d model
* X - transformations on the model, including projection on the surface
* scr2  - found points on image

i.e. we are trying to find those X, where error will be minimum.
It will be our face coefficients.
 
## Drawing effects

Knowing face orientation and facial expression.
We could write result using OpenGL(Open Graphics Library).
It's a cross-platform language to use GPU(Graphic Processing Units) for 2d and 3d graphics.
It can make a lot of simple computations in parallel.
The main term is shaders.
Shaders - it's programs that applied on GPU.
There are two main shaders: vertex and fragment.
Vertex shaders for doing computation for points.
And fragments shaders for providing colours for each pixel.
Shaders could use textures.
Texture is 2d image for the shader, examples - masks for face and input camera preview where we apply texture.

## Conclusion

In overall, to build this effect we had to use:
* camera - get image from camera
* OpenCV - library to find face and computation for face orientation and expression
* dlib - library to find point on the face
* OpenGL - drawing effects using Android's GPU

And as result we have Android application that can do all that things.

Sources of the library [https://github.com/oleg-sta/VrFace](https://github.com/oleg-sta/VrFace)

Examples of using library:
* [https://github.com/oleg-sta/FaceMaskExample](https://github.com/oleg-sta/FaceMaskExample)
* [https://github.com/oleg-sta/Masks](https://github.com/oleg-sta/Masks)
* [https://github.com/oleg-sta/MakeUp](https://github.com/oleg-sta/MakeUp)