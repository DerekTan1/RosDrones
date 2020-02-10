# TurtleBot Reflective Tape Scan and Move

The TurtleBot will rotate 360 degrees until it detects reflective tape. When reflective tape is detected, the TurtleBot moves toward the reflective tape.

## Key Things to Note
When using the relective tape, the lidar distances are thrown off, hence, using lidar for obstacle avoidance is not possible. Be careful that the TurtleBot does not run into the wall. Use control + c to exit the script and halt the TurtleBot.

## Video
https://drive.google.com/file/d/1KmbKrSSngbihidJLRRpp0feHjWF2djWp/view?usp=sharing

## Setup
First, tape a strip of reflective tape about 10 inches long to the wall. Make sure that the tape is level with the Lidar sensor.
Then, copy the .py file into a package. Visit http://wiki.ros.org/ROS/Tutorials/CreatingPackage for help with this.

You are able to run the demo using:

```
$ rosrun <name_of_your_package> reflectiveTape.py
```

## reflectiveTape.py
```Ruby
#!/usr/bin/env python

import rospy
import math

from geometry_msgs.msg import Twist
from sensor_msgs.msg import LaserScan
from std_msgs.msg import Float32
from std_msgs.msg import Int8
from geometry_msgs.msg import PoseStamped
'''from sensor_msgs.msg import Float32'''

import sys, select, termios, tty

global averageCenter
averageCenter=0

global intensityLocation
intensityLocation=-1

global intensity
intensity=-1

def shutdown_sequence():
    twist = Twist()
    twist.linear.x = 0.0; twist.linear.y = 0.0; twist.linear.z = 0.0
    twist.angular.x = 0.0; twist.angular.y = 0.0; twist.angular.z = 0.0
    pub.publish(twist)

def callback(data):
    global averageCenter
    #rospy.loginfo("ranges %f", data.ranges[0])
    sum =0
    sum2 =0
    for i in range(0,5):
        sum=sum+data.ranges[i+355]
        sum2=sum2+data.ranges[i+1]
    averageCenter=(sum+sum2)/10

def statusCallback(msg):
    global status
    status=msg.data

def positionCallback(msg):
    global xpos
    xpos=msg.pose.position.x

def intensityLocationCallback(msg):
    global intensityLocation
    global intensity
    for x in range(0,359):
        intensity=msg.intensities[x]
        if int(msg.intensities[x])>4000:
            intensityLocation=x
            print(intensityLocation)
    
def scan():
    global averageCenter
    global status
    rate = rospy.Rate(30)
    twist = Twist()
    turning=False
    count=0
    boolstatus=False
    while not rospy.is_shutdown():
        if intensityLocation >0:
            boolstatus=True
        if boolstatus==True:
            print ("averageCenter: "+str(averageCenter))
            if .1< averageCenter <.30:
                twist.linear.x = 0; twist.linear.y = 0; twist.linear.z = 0
                twist.angular.x = 0; twist.angular.y = 0; twist.angular.z = 0
                pub.publish(twist)
            if (intensityLocation<4 or intensityLocation>356):
                twist.linear.x = .75; twist.linear.y = 0; twist.linear.z = 0
                twist.angular.x = 0; twist.angular.y = 0; twist.angular.z = 0
                pub.publish(twist)
            elif 4<=intensityLocation<180:
                twist.linear.x = 0; twist.linear.y = 0; twist.linear.z = 0
                twist.angular.x = 0; twist.angular.y = 0; twist.angular.z = .3
                pub.publish(twist)
            elif 180<intensityLocation<=356:
                twist.linear.x = 0; twist.linear.y = 0; twist.linear.z = 0
                twist.angular.x = 0; twist.angular.y = 0; twist.angular.z = -.3
                pub.publish(twist)   
        elif boolstatus==False:
            if (averageCenter==0.0):
                pass
            elif (averageCenter>.4 and turning==False and count==0):
                twist.linear.x = .75; twist.linear.y = 0; twist.linear.z = 0
                twist.angular.x = 0; twist.angular.y = 0; twist.angular.z = 0
                pub.publish(twist)
            else:
                turning=True
                if (count<=10 and turning==True):
                    twist.linear.x = 0; twist.linear.y = 0; twist.linear.z = 0
                    twist.angular.x = 0; twist.angular.y = 0; twist.angular.z = 1.35
                    pub.publish(twist)
                    count=count+1
                    print(count)
                    if (count>10):
                        count=0
                        turning=False
        rate.sleep()

def move():
    global averageCenter
    global intensityLocation
    global intensity
    rate = rospy.Rate(10) # 10hz
    twist = Twist()
    count=0
    turning=False
    while not rospy.is_shutdown():
        rate.sleep()

if __name__ == '__main__':
    rospy.on_shutdown(shutdown_sequence)
    '''turtlebot_teleopr'''
    pub = rospy.Publisher('cmd_vel', Twist, queue_size=5)
    sub = rospy.Subscriber("scan", LaserScan, callback)
    sub = rospy.Subscriber("scan", LaserScan, intensityLocationCallback)
    sub = rospy.Subscriber("/visp_auto_tracker/status", Int8, statusCallback)
    sub = rospy.Subscriber("/visp_auto_tracker/object_position", PoseStamped, positionCallback)
    rospy.init_node('test', anonymous=True)
    try:
         scan()
    except rospy.ROSInterruptException:
         pass
```

### Authors

* Created by Derek Tan
