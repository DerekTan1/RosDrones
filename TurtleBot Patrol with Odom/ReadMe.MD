# TurtleBot Patrol with Odom

The TurtleBot will move foward for a set amount, turn 180 degrees, and repeat.

## Key Things to Note
This is not the most efficient and accurate way to turn 180 degrees as the logic to check for the Odom compass position is very inefficient. The compass readings on the TurtleBot are inherently glitchy can also be thrown off depending on any nearby metal. The inaccuracies can be seen in the video. To compensate for this edit what odomW is subtracted by to offset the Odom until the TurtleBot turns 180 degrees. The lines to edit are shown below.

```
if (odomW-1.0-.05<initW<odomW-1.0+.05):
```
```
if (odomW+1.0-.05<initW<odomW+1.0+.05):
```

## Video
https://drive.google.com/file/d/1FtQJuc3PHNmWy92cNZAyF9jxQqcr-8NQ/view?usp=sharing

## Setup
First copy the .py file into a package. Visit http://wiki.ros.org/ROS/Tutorials/CreatingPackage for help with this.

You are able to run the demo using:

```
$ rosrun <name_of_your_package> patrol.py
```

## patrol.py
```ruby
#!/usr/bin/env python

import rospy
import math
from geometry_msgs.msg import Twist
from geometry_msgs.msg import TransformStamped
from sensor_msgs.msg import LaserScan
from nav_msgs.msg import Odometry

global timer
global odomW

timer= 0
odomW = 0

def shutdown_sequence():
    twist = Twist()
    twist.linear.x = 0.0; twist.linear.y = 0.0; twist.linear.z = 0.0
    twist.angular.x = 0.0; twist.angular.y = 0.0; twist.angular.z = 0.0
    pub.publish(twist)

def OdomCallBack(msg):
    global odomW
    odomW=round(msg.pose.pose.orientation.w,4)


if __name__=="__main__":
   

    rospy.init_node('turtle_chaser')
    rospy.on_shutdown(shutdown_sequence)

    pub = rospy.Publisher('cmd_vel', Twist, queue_size=10)
    subscriber = rospy.Subscriber('vicon/WANDV2/WANDV2', TransformStamped, GoalCallBack)
    subscriber = rospy.Subscriber('odom', Odometry, OdomCallBack)

    set_rate = 10
    dt = 1.0/set_rate
    rate = rospy.Rate(set_rate)

    # PID
    Kp = 1.1
    Ki = 0.0
    Kd = 0.0

    previous_error = 0
    integral = 0
    cycleCount = 0

    while (timer<1500):
        twist = Twist()
        twist.linear.x = 1.0
        twist.linear.y = 0.0
        twist.linear.z = 0.0

        twist.angular.x = 0.0
        twist.angular.y = 0.0
        twist.angular.z = 0.0
        pub.publish(twist)
        timer=timer+1
        if (timer==50):
            initW=odomW
            #print("initial W: "+str(initW))
        if (timer>=50):
            #print("W: " + str(odomW))
            twist = Twist()
            twist.linear.x = 0.0
            twist.linear.y = 0.0
            twist.linear.z = 0.0
            twist.angular.x = 0.0
            twist.angular.y = 0.0
            twist.angular.z = 0.75
            pub.publish(twist)
            if (odomW>0):
                if (odomW-1.0-.05<initW<odomW-1.0+.05):
                    cycleCount=cycleCount+1
                    #print("cycle: "+ str(cycleCount))
                    if (cycleCount==1):
                        timer=0
                        cycleCount=0
            if (odomW<0):
                if (odomW+1.0-.05<initW<odomW+1.0+.05):
                    cycleCount=cycleCount+1
                    #print("cycle: "+ str(cycleCount))
                    if (cycleCount==1):
                        timer=0
                        cycleCount=0
        if (timer>800):
            timer=0
        rate.sleep()
```

### Authors

* Created by Derek Tan
