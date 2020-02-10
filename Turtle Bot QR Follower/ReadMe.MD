# Turtle Bot QR Follower

TurtleBot 1 will have a QR code attached to it and be controlled through Tele-Operations. TurtleBot 2 will follow TurtleBot1.

## Key Things to Note
This code uses the same logic as the QR Scan and Move code combined with the TurtleBot Basic Lidar Obstacle Avoidance code. After the code is loaded, control TurtleBot 1 using the a,w,s,d, and x keys. If you turn too quickly the following TurtleBot will lose track of the QR code.

## Video
https://drive.google.com/file/d/1sF-5vlOWY3TK4JfenZ_sCKrUjebrjfof/view?usp=sharing

## Setup
First copy the .py file into a package. Visit http://wiki.ros.org/ROS/Tutorials/CreatingPackage for help with this.

To load multiple TurtleBots onto the same Roscore click [here.](https://github.com/hildebrandt-carl/Autosoft_Lab/blob/master/Posts/ConnectMultipleTB.MD)

You are able to run the demo using:

```
$ ROS_NAMESPACE=tb3_0 roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch multi_robot_name:=tb3_0
```
```
$ ROS_NAMESPACE=tb3_1 rosrun <name_of_your_package> qrTowardPatrol.py multi_robot_name:=tb3_1
```

## qrTowardPatrol.py
```ruby
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

global status
status=0

global xpos
xpos=0

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

def scan():
    global averageCenter
    global status
    rate = rospy.Rate(30) # 10hz
    twist = Twist()
    while not rospy.is_shutdown():
        print(averageCenter)
        if .1< averageCenter <.32:
            break
        if (.015<xpos<.025):
            twist.linear.x = 1; twist.linear.y = 0; twist.linear.z = 0
            twist.angular.x = 0; twist.angular.y = 0; twist.angular.z = 0
            pub.publish(twist)
        elif xpos<=0.015:
            twist.linear.x = 0; twist.linear.y = 0; twist.linear.z = 0
            twist.angular.x = 0; twist.angular.y = 0; twist.angular.z = .1
            pub.publish(twist)
        elif xpos>=.025:
            twist.linear.x = 0; twist.linear.y = 0; twist.linear.z = 0
            twist.angular.x = 0; twist.angular.y = 0; twist.angular.z = -.1
            pub.publish(twist)   

        rate.sleep()

if __name__ == '__main__':
    rospy.on_shutdown(shutdown_sequence)
    '''turtlebot_teleopr'''
    pub = rospy.Publisher('cmd_vel', Twist, queue_size=5)
    sub = rospy.Subscriber("scan", LaserScan, callback)
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
