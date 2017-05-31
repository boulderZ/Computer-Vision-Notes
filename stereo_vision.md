## Stereo Vision Notes
This serves to document some of my study of computer vision. Meant to be used as a reference for myself but others may find it useful. I am only highlighting areas for my own learning and so it is not comprehensive and skips over a lot of required background that I already have. Below figures and text are taken from multiple sources listed below.

[figure1]: ./projection_coordinate_system_big.png
[figure1a]: ./Perspective_Projection.png
[Rigid Transformation]: ./Rigid_Transformation.png
[Rigid Transformation Homgenous]: ./Rigid_Transformation_Homogenous.png
[Rotation Matrix R]: ./Rotation_Matrix_R.png
[figure2]: ./Stereo_geometry_depth.png

[basic w/o depth in Matlab-mccormick](http://mccormickml.com/2014/01/10/stereo-vision-tutorial-part-i/)

[Udacity Intro to Computer Vision-ud810](https://www.udacity.com/course/introduction-to-computer-vision--ud810)

images/ud810/
### Perspective Projection
To describe projection conveniently in math terms, we place the center of projection at the origin and put the image plane in front of it to avoid dealing with the fact that a real camera inverts the image. This also means that y direction is positive in up direction which is opposite of normal image processing.

<img src="./images/ud810/projection_coordinate_system_big.png" width="500" height="250"/>

Next figure shows how to use matrix formulation with homogeneous coordinates.

<img src="./images/ud810/Perspective_Projection.png" width="500" height="250"/>

Where $(u,v)$ represents the coordinates in the image of some point $(x,y,z)$ out in the world projected through a projection with focal length of $f$. So we convert to $(u,v)$ when we need to work with an image.

### Extrinsic and Intrinsic Parameter matrices
#### Extrinsic
Extrinsic parameter matrix transforms from world view to camera view.

<img src="./images/ud810/Rigid_Transformation.png" width="500" height="250"/>

Then using homogeneous coordinates

<img src="./images/ud810/Rigid_Transformation_Homogenous.png" width="500" height="250"/>

Examples of Rotation Matrx R:

<img src="./images/ud810/Rotation_Matrix_R.png" width="500" height="250"/>

<br><br>

<img src="./images/ud810/World_to_Camera.png" width="500" height="250"/>

#### Intrinsic
Intrinsic is from 3D coordinates in camera frame to the 2D image plane via projection.

<img src="./images/ud810/Intrinsic_Matrix.png" width="500" height="250"/>

Then can simplify by dropping last column and different notation. Note that this version has 5 DOF because of unsquare pixels, off center optical axis, and skew.
<img src="./images/ud810/Intrinsic_Matrix_Reduced.png" width="500" height="250"/>

If we make enough handy assumptions we can get to the following matrix for K:

<img src="./images/ud810/Intrinsic_Matrix_Simple.png" width="500" height="250"/>

#### Combined Extrinsic and Intrinsic, Camera Parameters

<img src="./images/ud810/Combined_Extrinsic_Intrinsic.png" width="500" height="250"/>

And moving to using M matrix:

<img src="./images/ud810/Extrinsic_Intrinsic_M.png" width="500" height="250"/>

Another way to write it out:

<img src="./images/ud810/More_Camera_Matrix.png" width="500" height="250"/>

A camera and it's matrix M:

<img src="./images/ud810/Camera_Parameters.png" width="500" height="250"/>

Where the blue parameters are extrinsics and red are intrinsics.

Then showing all the effects
<img src="./images/ud810/Cumulative_Camera_Parameters.png" width="500" height="250"/>

#### Stereo system geometry for depth measurement
The geometry for co-planar images (taken from [Udacity Intro to Computer Vision-ud810](https://www.udacity.com/course/introduction-to-computer-vision--ud810) is shown here

<img src="./images/ud810/Stereo_geometry_depth.png" width="500" height="250"/>

The  term  $x_l-x_r$ is known as disparity.

### Calibrating cameras

### Image to Image projections

<img src="./images/ud810/General_Projective_Transform.png" width="500" height="250"/>

General projective transform is also known as "Homography"

### Geometry to Algebra
<img src="./images/ud810/Geometry_to_Algebra.png" width="500" height="250"/>

Matrix form of cross product:

<img src="./images/ud810/Matrix_Form_Cross_Product.png" width="500" height="250"/>

<img src="./images/ud810/Cross_Product_Notation.png" width="500" height="250"/>

Essential Matrix:

<img src="./images/ud810/Essential_Matrix.png" width="500" height="250"/>

Essential matrix with parallel cameras:

<img src="./images/ud810/Essential_Matrix_Parallel_1.png" width="500" height="250"/>
<img src="./images/ud810/Essential_Matrix_Parallel_2.png" width="500" height="250"/>
<img src="./images/ud810/Essential_Matrix_Parallel_3.png" width="500" height="250"/>

### Fundamental Matrix
This section goes over case where the cameras are uncalibrated and how you can derive the Fundamental matrix.

Start with "weak calibration" which means there is no skew. Modern cameras have near zero skew and no radial distortion so we ignore it for these calculations.

<img src="./images/ud810/F1-Weak_Calibration.png" width="500" height="250"/>

Now the K intrinsic matrix is invertible so using left and right camera points, you can get the camera points (homogeneous along ray) from the image points and using proper of the Essential matrix can show below

<img src="./images/ud810/F2-Uncalibrated_Case.png" width="500" height="250"/>

Then can re-write as:

<img src="./images/ud810/F3-Fundamental_Matrix.png" width="500" height="250"/>

Properties of Fundamental Matrix:

<img src="./images/ud810/F4-Properties_Fundamental.png" width="500" height="250"/>

Likewise you can reverse above to get l' = F^Tp from p^TFp'.
Then you can get the epipoles from F as:

<img src="./images/ud810/F5-Epipoles.png" width="500" height="250"/>

Then to compute F

<img src="./images/ud810/F6-Compute_F.png" width="300" height="100"/>
<br>


<img src="./images/ud810/F7-Compute_F_multiple.png" width="500" height="250"/>

There is a problem with solving above equations directly with SVD. The resulting epipolor lines will not converge at same point because there was no restriction on the rank of F which must be two as shown below.
<img src="./images/ud810/F8-Rank_of_F.png" width="500" height="250"/>

So to fix this issue, you first solve for F using SVD. Then decompose F using SVD, remove smallest eigenvalue and get F':
<img src="./images/ud810/F9-Fix_linear_solution.png" width="500" height="250"/>
<img src="./images/ud810/F10-Estimate_new_F.png" width="300" height="100"/>

Summary

<img src="./images/ud810/Summary_Geometry_Algebra.png" width="500" height="250"/>

#### Basic Idea of extracting depth from stereo images
The difference in horizontal position of the same object (feature) taken from two views is proportional to the depth of that object. Search for matching patches and calculate the distance.

#### Image Rectification
Image Rectification is process of insuring that the two stereo images have been transformed such that you only have to search horizontally by row to find matching patches.

1) Does this step introduce distortion? Guess is yes.

2) What is complexity cost of rectification vs full search?

#### Search Range and Direction
To reduce complexity, only search up to maximum expected disparity. General case of search direction is that you can have disparity of both positive and negative values if the stereo cameras are not aligned. If aligned, you can save time and search in only one direction.
