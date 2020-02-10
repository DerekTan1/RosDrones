# TurtleBot Basic Lidar Obstacle Avoidance

The TurtleBot will move straight until an object is .5 meters away from it. It will then turn left approximately 90 degrees and repeat this cycle.

## Key Things to Note
The amount that the TurtleBot turns will be dictated by the count variable in the move() function. To increase how long the TurtleBot turns for either vary the count variable.


## Setup
First copy the .py file into a package. Visit http://wiki.ros.org/ROS/Tutorials/CreatingPackage for help with this.

You are able to run the demo using:

```
$ rosrun <name_of_your_package> lidarObstacleAvoid.py
```

## lidarObstacleAvoid.py
```ruby
#!/usr/bin/env python

import rospy
import math

from geometry_msgs.msg import Twist
from sensor_msgs.msg import LaserScan
from std_msgs.msg import Float32
'''from sensor_msgs.msg import Float32'''

import sys, select, termios, tty

global averageCenter
averageCenter=0

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

def move():
    global averageCenter
    rate = rospy.Rate(10) # 10hz
    twist = Twist()
    count=0
    turning=False
    while not rospy.is_shutdown():
        #print(averageCenter)
        if (averageCenter==0):
            pass
        elif (averageCenter>.5 and turning==False):
            twist.linear.x = 1; twist.linear.y = 0; twist.linear.z = 0
            twist.angular.x = 0; twist.angular.y = 0; twist.angular.z = 0
            pub.publish(twist)
        else:
            turning=True
            if (count<=6 and turning==True):
                twist.linear.x = 0; twist.linear.y = 0; twist.linear.z = 0
                twist.angular.x = 0; twist.angular.y = 0; twist.angular.z = 1.5
                pub.publish(twist)
                count=count+1
                print(count)
                if (count>6):
                    count=0
                    turning=False
        rate.sleep()

if __name__ == '__main__':
    rospy.on_shutdown(shutdown_sequence)
    '''turtlebot_teleopr'''
    pub = rospy.Publisher('cmd_vel', Twist, queue_size=5)
    sub = rospy.Subscriber("scan", LaserScan, callback)
    rospy.init_node('test', anonymous=True)
    try:
        move()
    except rospy.ROSInterruptException:
        pass

```

### Authors

* Created by Derek Tan
