# Computer Graphics presentation

December 8, 2017

## Features

A user is able to view and rotate a fully illuminated 3D model of the Moon, with or without Google Cardboard.

## Instructions

Navigate to [dawneraq.github.io/Lune](https://dawneraq.github.io/Lune/) on either a desktop or mobile device.

If you have a Cardboard viewer, place your phone into it in landscape mode. You'll get the best results if you rotate your phone *counterclockwise* from portrait orientation.

If you don't have a viewer, you can still examine the model by moving the scene's camera around based on your phone's orientation. You can also drag your finger along the screen to rotate the camera, similar to the mouse controls on desktop.

Crossing your eyes to achieve stereoscopic 3D is possible, but not recommended.

## Major design decisions

I chose to design my term project around Google Cardboard because I got a free viewer from work over the summer. The example apps are pretty mindblowing, and showcase the technology's capability in a variety of ways.

For the rendering and camera controls, I used ThreeJS, an open-source JavaScript library that acts as a wrapper around a WebGL renderer. Cardboard had ThreeJS starter code at [vr.chromeexperiments.com](https://vr.chromeexperiments.com/).

ThreeJS also provides loaders for the 3D model file formats I used.

Instead of a topographically accurate 3D model of the Moon, the model that's currently displayed is one created by a free3d.com user in Maya. It consists of:

1. An .obj file representing the vertices of a perfect sphere, and
1. An .mtl file with references to two .jpgs, a 2500x1250px texture map and a bump map, each of which is projected onto the sphere.

The result of projecting rectangles onto a sphere is that the surface texture degenerates at the poles, which is not a huge drawback. It's certainly better than no visible model at all.

## Difficulties encountered

After getting Chrome Experiments' Cardboard boilerplate code set up with the free3d model, I loaded into the scene [a 3D shapefile model of the Moon](https://pubs.usgs.gov/of/2006/1367/derived/) released in 2005 by the USGS. For one reason or another, I could not get this model to display, even though there were no JavaScript errors.

The biggest challenge I ran into was getting ThreeJS' DeviceOrientationControls--which control the scene's camera--to respond correctly to changes in phone orientation. According to the [standard API for communicating device orientation information](https://developers.google.com/web/fundamentals/native-hardware/device-orientation/), the x and y axes are in the plane of the screen, positive towards the right and towards the top, respectively. And the z axis is normal to the screen, positive extending outward.

With the phone positioned in counterclockwise-landscape orientation, the y and z axes are swapped. It took quite a bit of tinkering to figure out that to make the camera movement work correctly, I had to change the sequence of rotation axes for the Tait-Bryan angles in DeviceOrientationControls from Y-X'-Z'' to X-Y'-Z''.

```javascript
// js/third-party/threejs/DeviceOrientationControls.js

this.alpha = deviceOrientation.alpha ?
  THREE.Math.degToRad(deviceOrientation.alpha) : 0; // Z
this.beta = deviceOrientation.beta ?
  THREE.Math.degToRad(deviceOrientation.beta) : 0; // X'
this.gamma = deviceOrientation.gamma ?
  THREE.Math.degToRad(deviceOrientation.gamma) : 0; // Y''
this.orient = screenOrientation ?
  THREE.Math.degToRad(screenOrientation) : 0; // O

// The angles alpha, beta and gamma
// form a set of intrinsic Tait-Bryan angles of type Z-X'-Y''

// 'ZXY' for the device, but 'XYZ' for us
euler.set(this.alpha, this.beta, - this.gamma, 'XYZ');
```