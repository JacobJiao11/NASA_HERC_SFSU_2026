By: Therese Caburnay
Purpose: Provide code for camera and LiDAR sensor for SFSU NASA HERC Rover. 

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Camera: Raspberry Pi Camera Module
- Able to capture and save image. Apply date and time of when the image was taken to the image. 
- Open image to apply filter of red, green, or blue color.
- Save new photo if color filter was applied 
- Assist LiDAR sensor in identifying objects in path

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

LiDAR Sensor: 
- Able to identify a object in path and measure distance.
- Be able to return that distance. 

== INFORMATION FOR LIDAR: ==
The sensor contiously procdes a 360 degree scan of its surrounding enviroment. The scan information has 3 parameters;
 (angle, distance, intensity). For example, (0.7 degrees, 337mm) meaning  at 0.7 degrees there is something 337 mm away. 
 Max range is ~12m. Scans are 10Hz.

Data Organization:
- Serial Communication
- Baud Rate: 230400
- 12 measurement points per packet
- Distance in mm
- Angels is in degrees
- Intensity is relflected laser signal

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Completed as of 10/30/2026 @ 8:30PM:
Imported Picamera2, Pillow, and time library
Created functions to apply filters
Configure camera
    - able to caputre and save image
What is left to do:
Import LiDAR library
Have LiDAR Measure distance and return it 
Have camera help LiDAR in identify objects
Apply date and time to caputred images
Questions:
Do I need a function to capture image? Or is an image caputred every few seconds 