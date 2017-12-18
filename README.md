# Udacity-P4--Advanced-Lane-Finding
## Calibrate camera using chessboard pictures
The calibration camera matrix and distortion coefficients are calculated through a cheeseboard picture taken by the camera. The corners on the chessboard are detected, and then the distorted images are then undistorted by putting the corners onto the same plane.

## Line detection using HLS, Color and Gradient
I have tried to use HLS and RGB thresholds, as well as gray/color x gradient to detect lines. I have fine-tuned the thresholds to better pick up the lanes and to minimize the noises.

- For HLS threshold, S does a good job in detecting both yellow and white lines, however, when there are shadows, S tends to pick up noises. L threshold picks up white lines very well, but is totally blind to yellow lines. H is not helpful in detecting lines.

- For RGB thresholds, R performs better than B and G, and is able to pick up both yellow and white lines.

- I have also used LAB threhold. The B channel picks up yellow lines very accurately, but completely ignores white lanes.

- I have also applied x direction gradient thresholds on both RGB channels and gray images. B and gray gradient are performing better than R and G gradient, however, the direction gradient picks up a lot of other noises, so I did not end up using them.

## Perspective Transform
I have performed perspective transform to convert the 3D image into 2D image to provide a bird's eye view of the lanes. The transform coordinates are determined by trial and error, so that the straight lanes in the test images are transformed into parallel lines.

In the beginning, I have only selected the bottom part of the curves. That part of the lines tend to be more straight, and thus I lose the curvature information. In order to better capture the curvature, I have included the curves higher up.

## Lane Detection
I have implemented a sliding window methods to detect lines from the image. This method uses a histogram of detected pixels to locate the base of the lanes. From there, sliding windows are constructed towards the intercept point to detect the remaining relevant pixels. A polynomial function is then fitted seperately to the detected points for the left and right lanes.

If there are saved lines detected in the previous frame, I started the line search from the previously fitted lines in order to save searching time.

## Draw Polygon and calculate curvatures
I drew a polygon in the area between two detected lanes, and calculated the curvature of the lanes and distance from lane center.

To calculate the curvature, I used the radius of curvature formula from here: https://www.intmath.com/applications-differentiation/8-radius-curvature.php. The formula uses the first and second derivatives of the polynomial to calculate the radius. The coordinates are converted to meters to fit the polynomials in the real world space.

To find out the distance of the car from the lane center, I first calculated the center of the fitted left and right lanes as the midpoint of the bottom points. Then I located the center of the image. The difference between the two is the offset from the center. 

## Create a class for lane lines
I have created a class called Line in order to record fitted lines in the previous frame. There are several reasons for using a class:
- In case of a bad frame where the model is not able to detect lines properly, we can still pick up the fits from the previous frames.
- to smooth out the fit across different frames by averaging the past fit parameters
- if there are saved fits from the previous frames, start the line search from the previous fitted lines, instead of searching from scratch using sliding window 
  
## Create pipeline function 
In this step, I created a pipeline function to run through all the steps described above. The output image has a polygon highlighted in between the two lanes. 

I ran the pipeline function through the test images. Since these test images are randomly selected at random intervals from the original video, they are not adjacent frames. The images below show how the smoothing function works, whereby it averages the previous frame's fitted curvatures with the current frame, hence the misfit and incorrect curvatures shown. Once we apply this smoothing function to our full video, we observe the merits of the averaging scheme, where the image frames are continuous.

## Run pipeline on video

## Improvements
- try to improve the line finding process. Now the curvatures tend to oscilate when there are shadows. 
- automated ways of defining the coordinates for perspective transform. The final curvature results are quite sensitive to how I define the transform coordinates. If I choose coordinates lower towards the bottom, only the lower part of the lanes are studied, which tends to be straight and the curvatures are higher. If I choose the coordinates upper towards the intercept, the shape of the lanes will be better captured and the curvatures are tend to be smaller. 
- check if the two detected lanes are more or less parallel. Ideally, the two fitted lines should be parallel to each other. Again it is pretty sensitive to the perspective transform.
- find ways to produce more stablized curvatures.
