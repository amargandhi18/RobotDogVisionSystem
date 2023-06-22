# RobotDogVisionSystem
Code for a QR code detection algorithm which computes the real time distance of the QR code relative to the robot. This uses python and openCV.

Description:

The vision system of the quadruped robot is included in this project to follow an operator through the use of a QR code. The robot will track the QR code by computing its distance and relative area. This is done in python through the use of the OpenCV library. Python code was developed to capture a QR code in a video frame and to compute a distance and direction of the QR code relative to the camera, which in this case was a laptop webcam. 

The methodology in attaining this algorithm goes as follows. The first step was to derive a formula to calculate the distance of the QR code relative to the camera. After some research it was noted that the focal length of the utilized camera is a key parameter in doing so. The focal length can allow the code to perceive the camera’s depth and field of view. To relate the focal length and distance, the triangle similarity can be used. Triangle similarity involves an object with a known width being placed at a known distance. It also takes the apparent width of the object in an image capture. This measurement is in pixels. Thus, the focal length can be calculated:

Focal Length = (Pixel Width * Known Distance) / Known Width

After focal length has been calculated, the distance can be computed by multiplying it with the known width. This is then divided by the pixel width, to get a distance measurement.

Distance = (Known Width * Focal Length) / Pixel Width 39

Therefore, in order for the distance to be computed, the QR code first needs to be placed at a known distance from the webcam.
The next part involves capturing the QR code in the frame to attain key attributes such as pixel coordinates of the bounding box and the pixel width. Significant modifications were made to this process as last semester’s method was inaccurate and unstable. Previously the OpenCV code was utilizing functions such as “Gaussian Blur” and “Canny Edges” in capturing any rectangular box in its field of view. Although the QR code was contained in a rectangular piece of paper, background objects were still detected thus accounting for inaccurate measurements. This is because these objects were momentarily taken into the formula that calculates the distance.

A solution was found to eliminate this problem. It was identified that in order to increase accuracy, the QR code has to be the only object the camera is identifying so that the robot will follow only the QR code. This was done through another open-source library “pyzbar”. Within this library, there is a decode function that is mainly used to identify and return certain attributes of a barcode or QR code. This is highly useful because a certain parameter that is required for the distance calculator is the perceived width of the QR code in pixels within a frame. The decode function satisfies this requirement by returning a bounding box, a 2D array which contains both height and width of the QR code in pixels. This width value was then taken into the distance formula. The end results proved to be significantly more accurate than the previous method.

Once the distance aspect of the vision system was complete and stable, the next task involved developing code in order for the robot to follow a certain direction to reach the QR code. In the code, the QR code is first identified in the frame using a decode function similar to what was explained above. Along with this, the centre point of the frame is extracted to generate a left and right section for the robot to follow. The QR code’s centre point is used as its location. This is derived from averaging opposite polygon points. The difference between the QR code’s centre and the center point of the frame is computed to determine if the QR code is in the left, centre or right in the video frame. The end goal will be for the robot to follow each of these depending on the QR code’s location in the video feed.

Once both aspects of the vision system were completed, the two scripts were then integrated into one combined script, returning a magnitude and a direction (L or R). It was then time to apply this code to the quadruped for testing. In order for this to be ideal, two things would be required. A raspberry pi would be needed to process the run the vision code along with a connected webcam. Another alternative was to use the already existing camera within the quadruped itself, however the camera quality was not up to  expectations. 

After loading the code onto the raspberry pi the algorithm was tested with the logitech c270 webcam. Some slight modifications were made in order for the code to cooperate with the pi such as changing the video capture source to the webcam and setting the new calibration image since a new camera was involved. After this, the code proved to be a success as a distance measurement and a direction containing ‘L’ or ‘R’ were generated. The code on the raspberry pi then communicates with the arduino which is also connected with the mini joystick enabling it to move in 4 directions (F, B, L, R) depending on the reference voltage. 

After testing, the vision code allowed the quadruped to move, but it was a bit unstable in terms of left and right direction. It was noted that the left and right boundaries in the vision code were too large allowing for a minimal forward threshold. It was decided that the left and right threshold margins were to be decreased, while increasing the forward threshold. This would be done by establishing three sections in the video frame, each one containing a left, forward and right boundary area. If the center point of the QR code falls in any of these boundaries, that respective boundary area would be returned. This allows for more accurate direction tracking since the quadruped will not always be moving left and right. 

In the code “w1” and “w2” are an integer containing a pixel value acting as the first third and second third of the video feed respectively. This allows for an if statement to declare more accurate thresholds that correspond with the video frame. In the code it goes as follows:
 
The first if statement denotes the first section in the video frame which normally would be left, but since the quadruped will use the webcam as a vision tool, this would be flipped to right instead. The next if statement covers the second section returning a ‘F’ string meaning forward. Finally the third section returns the ‘L’ string meaning left. All of these statements return the distance measurement as well. These strings are then encoded and serial written to the arduino to enable movement from the quadruped.






