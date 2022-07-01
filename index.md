# Moving ball tracking robot
In this project I made a robot that is tracking the ball of the specific color. 

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Sviatoslav Oleksiienko | Maharishi School | Software Engineering, Computer science, Programming | Incoming Senior

![Headstone Image](https://gdurl.com/0gHQ)

# Demo night presentation 
<iframe width="560" height="315" src="https://www.youtube.com/embed/KBhEHXrmlh8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>{:target="_blank" rel="noopener"}

# Final Milestone
To sum up, I want to establish what I have now.
1.	I studied image processing and applied it
2.	I worked with the motors and connected them to the system
3.	I have a working project
4.	I found the best way for the power supply of the project


<iframe width="560" height="315" src="https://www.youtube.com/embed/Pin8OPTUCe4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>{:target="_blank" rel="noopener"}

# Second Milestone
I had a different version of the power supply in this project. Firstly, I tried to power raspberry, motors, and ultrasonic sensors from the AA batteries. But, during long usage, I concluded that it is unstable and expensive (because isn’t rechargeable).
I ran into a lot of problems during the process of buying details. I am a Ukrainian who is temporarily staying in Turkey, which means I don’t know the language, and how ordering something is working here. When I ordered, I had a lot of problems. For instance, for the body of the robot, the shop didn’t send 1 screw and I had to put one motor on a scotch tape. Moreover, I needed to buy sandpaper to wider the place for the switcher of the power supply, because it was too small to fit it. And all of this I needed to describe to the support, who speaks Turkish. Etc., etc.


<iframe width="560" height="315" src="https://www.youtube.com/embed/JHK3ra_gFGU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>{:target="_blank" rel="noopener"}

# First Milestone
During the first milestone I was working on image processing. It is important to find the range of the colors in HSV pallete that suits exactly your object. You can do this with this code. 
```py
import cv2
import numpy as np

def nothing(x):
    pass

# Load image
video = cv2.VideoCapture(0)

# Create a window
cv2.namedWindow('image')

# Create trackbars for color change
# Hue is from 0-179 for Opencv
cv2.createTrackbar('HMin', 'image', 0, 179, nothing)
cv2.createTrackbar('SMin', 'image', 0, 255, nothing)
cv2.createTrackbar('VMin', 'image', 0, 255, nothing)
cv2.createTrackbar('HMax', 'image', 0, 179, nothing)
cv2.createTrackbar('SMax', 'image', 0, 255, nothing)
cv2.createTrackbar('VMax', 'image', 0, 255, nothing)

# Set default value for Max HSV trackbars
cv2.setTrackbarPos('HMax', 'image', 179)
cv2.setTrackbarPos('SMax', 'image', 255)
cv2.setTrackbarPos('VMax', 'image', 255)

# Initialize HSV min/max values
hMin = sMin = vMin = hMax = sMax = vMax = 0
phMin = psMin = pvMin = phMax = psMax = pvMax = 0

while(1):
    # Get current positions of all trackbars
    ret, frame = video.read()
    image = frame
    hMin = cv2.getTrackbarPos('HMin', 'image')
    sMin = cv2.getTrackbarPos('SMin', 'image')
    vMin = cv2.getTrackbarPos('VMin', 'image')
    hMax = cv2.getTrackbarPos('HMax', 'image')
    sMax = cv2.getTrackbarPos('SMax', 'image')
    vMax = cv2.getTrackbarPos('VMax', 'image')

    # Set minimum and maximum HSV values to display
    lower = np.array([hMin, sMin, vMin])
    upper = np.array([hMax, sMax, vMax])

    # Convert to HSV format and color threshold
    hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
    mask = cv2.inRange(hsv, lower, upper)
    result = cv2.bitwise_and(image, image, mask=mask)

    # Print if there is a change in HSV value
    if((phMin != hMin) | (psMin != sMin) | (pvMin != vMin) | (phMax != hMax) | (psMax != sMax) | (pvMax != vMax) ):
        print("(hMin = %d , sMin = %d, vMin = %d), (hMax = %d , sMax = %d, vMax = %d)" % (hMin , sMin , vMin, hMax, sMax , vMax))
        phMin = hMin
        psMin = sMin
        pvMin = vMin
        phMax = hMax
        psMax = sMax
        pvMax = vMax

    # Display result image
    cv2.imshow('image', result)
    cv2.imshow('result', result)
    if cv2.waitKey(10) & 0xFF == ord('q'):
        break

cv2.destroyAllWindows()
```
The good question will be: “How exactly the code works”? I will try to explain quickly.
At first, it takes an image from the Pi Cam, converts it to the HSV color space (which is better for image processing), and makes a mask from it.
After, it outlines the contours and calculates the area size, which allows me to understand whether it is moving toward the ball.
In addition, it assigns the coordinate axis to all this system to make it use the rotation to put the ball into the center of the Pi Cam’s image.


<iframe width="560" height="315" src="https://www.youtube.com/embed/4FLv4QYLK8A" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>{:target="_blank" rel="noopener"}
