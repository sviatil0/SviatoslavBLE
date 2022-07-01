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

![Project schematic](https://gdurl.com/KLgM)

# Full code
```py
from picamera.array import PiRGBArray     #As there is a resolution problem in raspberry pi, will not be able to capture frames by VideoCapture
from picamera import PiCamera
import RPi.GPIO as GPIO
import time
import cv2
import cv2 as cv
import numpy as np

#hardware work
GPIO.setmode(GPIO.BCM) 

TRIGGER_LEFT = 20      #Left ultrasonic sensor
ECHO_LEFT = 26


TRIGGER_RIGHT = 16      #Right ultrasonic sensor
ECHO_RIGHT = 19

TRIGGER_FRONT = 12 #Front ultrasonic sensor
ECHO_FRONT = 6

LEFT_M1= 14  #Left Motor
LEFT_M2=4

RIGHT_M1=18  #Right Motor
RIGHT_M2=17


# Working with ultrasonic sensors
GPIO.setup(TRIGGER_LEFT,GPIO.OUT)  # Trigger
GPIO.setup(ECHO_LEFT,GPIO.IN)      # Echo
GPIO.setup(TRIGGER_RIGHT,GPIO.OUT)  # Trigger
GPIO.setup(ECHO_RIGHT,GPIO.IN)

GPIO.output(TRIGGER_LEFT, False)
GPIO.output(TRIGGER_RIGHT, False)



def sonar(GPIO_TRIGGER,GPIO_ECHO):
      start=0
      stop=0

      GPIO.setup(GPIO_TRIGGER,GPIO.OUT)  # Trigger
      GPIO.setup(GPIO_ECHO,GPIO.IN)      # Echo
     
      GPIO.output(GPIO_TRIGGER, False)
     
      time.sleep(0.01)
           
      GPIO.output(GPIO_TRIGGER, True)
      time.sleep(0.00001)
      GPIO.output(GPIO_TRIGGER, False)
      begin = time.time()
      while GPIO.input(GPIO_ECHO)==0 and time.time()<begin+0.05:
            start = time.time()
     
      while GPIO.input(GPIO_ECHO)==1 and time.time()<begin+0.1:
            stop = time.time()
      elapsed = stop-start
      distance = elapsed * 34000
      distance = distance / 2
     
      print("Distance : ",distance)
      return distance



# Working with motors
GPIO.setup(LEFT_M1, GPIO.OUT)
GPIO.setup(LEFT_M2, GPIO.OUT)

GPIO.setup(RIGHT_M1, GPIO.OUT)
GPIO.setup(RIGHT_M2, GPIO.OUT)

def forward():
    GPIO.output(LEFT_M1, GPIO.LOW)
    GPIO.output(LEFT_M2, GPIO.HIGH)
    GPIO.output(RIGHT_M1, GPIO.LOW)
    GPIO.output(RIGHT_M2, GPIO.HIGH)
     
def reverse():
    GPIO.output(LEFT_M1, GPIO.HIGH)
    GPIO.output(LEFT_M2, GPIO.LOW)
    GPIO.output(RIGHT_M1, GPIO.HIGH)
    GPIO.output(RIGHT_M2, GPIO.LOW)
     
def rightturn():
    GPIO.output(LEFT_M1,GPIO.LOW)
    GPIO.output(LEFT_M2,GPIO.HIGH)
    GPIO.output(RIGHT_M1,GPIO.HIGH)
    GPIO.output(RIGHT_M2,GPIO.LOW)
     
def leftturn():
      GPIO.output(LEFT_M1,GPIO.HIGH)
      GPIO.output(LEFT_M2,GPIO.LOW)
      GPIO.output(RIGHT_M1,GPIO.LOW)
      GPIO.output(RIGHT_M2,GPIO.HIGH)

def stop():
      GPIO.output(LEFT_M2,GPIO.LOW)
      GPIO.output(LEFT_M1,GPIO.LOW)
      GPIO.output(RIGHT_M2,GPIO.LOW)
      GPIO.output(RIGHT_M1,GPIO.LOW)
     







#Image analysis work
def segment_colour(frame):    #returns only the red colors in the frame
    hsv_roi =  cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)


    lower_yellow = np.array([11,85,199])
    upper_yellow = np.array([179,255,255])

    mask = cv2.inRange(hsv_roi, lower_yellow, upper_yellow)

    # Doing to clear the white noises
    kern_dilate = np.ones((8,8),np.uint8)
    kern_erode  = np.ones((3,3),np.uint8)
    
    mask= cv2.erode(mask,kern_erode)      #Eroding
    mask=cv2.dilate(mask,kern_dilate)     #Dilating
    return mask


def find_blob(blob): #returns the red colored circle
    largest_contour=0
    cont_index=0
    contours, hierarchy = cv2.findContours(blob, cv2.RETR_CCOMP, cv2.CHAIN_APPROX_SIMPLE)
    for idx, contour in enumerate(contours):
        area=cv2.contourArea(contour)
        if (area >largest_contour) :
            largest_contour=area
           
            cont_index=idx
                              
    r=(0,0,2,2)
    if len(contours) > 0:
        r = cv2.boundingRect(contours[cont_index])
       
    return r,largest_contour


def target_hist(frame):
    hsv_img=cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
   
    hist=cv2.calcHist([hsv_img],[0],None,[50],[0,255])
    return hist

#CAMERA CAPTURE
#initialize the camera and grab a reference to the raw camera capture
camera = PiCamera()
camera.resolution = (160, 120)
camera.framerate = 16
rawCapture = PiRGBArray(camera, size=(160, 120))

# allow the camera to warmup
time.sleep(0.001)
 
# capture frames from the camera
for image in camera.capture_continuous(rawCapture, format="bgr", use_video_port=True):
      #grab the raw NumPy array representing the image, then initialize the timestamp and occupied/unoccupied text
      frame = image.array
      frame=cv2.flip(frame,1)
      # cv2.imshow('initiall image',frame)
      global centre_x
      global centre_y
      centre_x=0.
      centre_y=0.
      hsv1 = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
      mask_red=segment_colour(frame)      #masking red the frame
      loct,area=find_blob(mask_red)
      x,y,w,h=loct
      # cv2.imshow('masked image',mask_red)
     
      distanceR = sonar(TRIGGER_RIGHT,ECHO_RIGHT)
      distanceL = sonar(TRIGGER_LEFT,ECHO_LEFT)
      distanceC = sonar(TRIGGER_FRONT,ECHO_FRONT)
      if (w*h) < 10:
            found=0
      else:
            found=1
            simg2 = cv2.rectangle(frame, (x,y), (x+w,y+h), 255,2)
            centre_x=x+((w)/2)
            centre_y=y+((h)/2)
            cv2.circle(frame,(int(centre_x),int(centre_y)),3,(0,110,255),-1)
            centre_x-=80
            centre_y=6--centre_y
            print(centre_x,centre_y)
      initial=400
      flag=0       
      if(found==0):
            #if the ball is not found and the last time it sees ball in which direction, it will start to rotate in that direction
            if flag==0:
                  rightturn()
                  time.sleep(0.05)
            else:
                  leftturn()
                  time.sleep(0.05)
            stop()
            time.sleep(0.0125)
     
      elif(found==1):
            if(area<initial):
                  if(distanceC<10):
                        #if ball is too far but it detects something in front of it,then it avoid it and reaches the ball.
                        if distanceR>=8:
                              rightturn()
                              time.sleep(0.00625)
                              stop()
                              time.sleep(0.0125)
                              forward()
                              time.sleep(0.00625)
                              stop()
                              time.sleep(0.0125)
                              #while found==0:
                              leftturn()
                              time.sleep(0.00625)
                        elif distanceL>=8:
                              leftturn()
                              time.sleep(0.00625)
                              stop()
                              time.sleep(0.0125)
                              forward()
                              time.sleep(0.00625)
                              stop()
                              time.sleep(0.0125)
                              rightturn()
                              time.sleep(0.00625)
                              stop()
                              time.sleep(0.0125)
                        else:
                              stop()
                              time.sleep(0.01)
                  else:
                        forward()
                        time.sleep(0.00625)
            elif(area>=initial):
                  initial2=6700
                  if(area<initial2):
                        if(distanceC>10):
                              #it brings coordinates of ball to center of camera's imaginary axis.
                              if(centre_x<=-20 or centre_x>=20):
                                    if(centre_x<0):
                                          flag=0
                                          rightturn()
                                          time.sleep(0.025)
                                    elif(centre_x>0):
                                          flag=1
                                          leftturn()
                                          time.sleep(0.025)
                              forward()
                              time.sleep(0.00003125)
                              stop()
                              time.sleep(0.00625)
                        else:
                              stop()
                              time.sleep(0.01)

                  else:
                        time.sleep(0.1)
                        stop()
                        time.sleep(0.1)    
      rawCapture.truncate(0)  # clear the stream in preparation for the next frame
         
      if(cv2.waitKey(1) & 0xff == ord('q')):
            break

GPIO.cleanup() #free all the GPIO pins
```
| **Quantity & Part Name** | **Part Description** | **Reference Designators** | **Cost** | **Link** |
|:--:|:--:|:--:|:--:|:--|
| 1 Screw Driver Kit | Included different screws & screw driver | N/A | $5.94 | https://www.amazon.com/Small-Screwdriver-Set-Mini-Magnetic/dp/B08RYXKJW9/ | 
| 3 Ping Sensors | Used for distance readings | HC - SR04 | $12.99 | https://www.amazon.com/ELEGOO-HC-SR04-Ultrasonic-Distance-MEGA2560/dp/B01COSN7O6/ |
| 1 H-Bridge | Used to power robot motors | X | $8.99 | https://www.amazon.com/ACEIRMC-Stepper-Controller-2-5-12V-H-Bridge/dp/B0923VMKSZ/ | 
| 1 Pi-Cam | Used for object detection and filtering | N/A | $9.99 | https://www.amazon.com/Arducam-Megapixels-Sensor-OV5647-Raspberry/dp/B012V1HEP4/ |
| 1 Set Jumpter Wires | Used for transferring voltage from object to object | N/A | $6.98 | https://www.amazon.com/Elegoo-EL-CP-004-Multicolored-Breadboard-arduino/dp/B01EV70C78/ |
| 1 Sodlering Kit | Used to melt metal together with solder | N/A | $19.99 | https://www.amazon.com/Soldering-Iron-Kit-Temperature-Screwdrivers/dp/B07GJNKQ8W/ref=sr_1_4_sspa?crid=3SWW7HN9U1AF1&keywords=20+dollar+soldering+kit&qid=1656613484&sprefix=20+dollar+soldering+kit%2Caps%2C59&sr=8-4-spons&psc=1&smid=A2CEQAD2VNOS6B&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFBSkI4UURNT0tRSlomZW5jcnlwdGVkSWQ9QTAwMzg5ODIzVkk0Nk03V1pSTzFRJmVuY3J5cHRlZEFkSWQ9QTA5Nzk4MTIxTUNMRUUwMlpWMlk1JndpZGdldE5hbWU9c3BfYXRmJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ== | 
| 1 cheap wire with usb (You are welcome to buy the cheapest you find) | Will be cut in a half and used for powering motors | N/A | $4.99 | https://www.amazon.com/Blackberry-micro-Charge-Cable-Generic/dp/B0041I6I5I/ref=sr_1_19?crid=1QPQN5NNXAMO9&keywords=micro+usb+wire&qid=1656699383&sprefix=micro+usb+wi%2Caps%2C203&sr=8-19 |
| 1 Power Bank | Used to power raspberry pi | N/A | $9.65 | https://www.amazon.com/Emilykylie-Multicolor-Cylinder-Battery-Charging/dp/B08YN9TL6P/ref=sr_1_3?crid=IWWD3H3E5Y5L&keywords=battery%2Bbank%2Bcylinder&qid=1656613614&s=electronics&sprefix=battery%2Bbank%2Bcylinder%2Celectronics%2C52&sr=1-3&th=1 |
| 1 Raspberry Pi 4 | Main hub of robot for functionality | N/A | $124.99 | https://www.amazon.com/Raspberry-Model-2019-Quad-Bluetooth/dp/B07TD42S27/ |
| 1 Micro SD Card | Used to transfer Pi data | N/A | $8.00 | https://www.amazon.com/gp/product/B089VVP61W/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1 | 
| 1 Portable Card Reader | Used to read Micro SD Card | N/A | $12.99 | https://www.amazon.com/dp/B07ZKRM12C/ref=sspa_dk_detail_0?psc=1&pd_rd_i=B07ZKRM12C&pd_rd_w=PMv06&pf_rd_p=7771f1a2-d77a-4098-a19e-6d9a1e65f44d&pd_rd_wg=WZWTX&pf_rd_r=N0S0M3D2VYYXTFJS0CYA&pd_rd_r=cb58c4a4-5003-4566-a6c9-b82133f4f19d&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFWMUUwWElYR045V0UmZW5jcnlwdGVkSWQ9QTA2NzgwNTMxQUNMVFA5NUg5SUxJJmVuY3J5cHRlZEFkSWQ9QTA0NDU5OTczM1JBTVg1M1dINFhWJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ== |
| Raspberry Pi Power Supply | Used to power raspberry pi wired | N/A | $7.99 | https://www.amazon.com/Replacement-Raspberry-Pi-4-Supply-Charger-Adapter/dp/B094J8TK61/ |
