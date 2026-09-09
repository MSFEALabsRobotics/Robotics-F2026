

# Publishers, Subscribers, Services

## Timers, Calback Functions


```bash
# using timer in ros2 nodes

#create a timer that calls a certain function, each number of seconds
self.timer = self.create_timer(2, self.callbackfunction)

#this will call the callbackfucntion each 2 seconds
```

```python
# the python local time function can also be used to make the program wait
import time

time.sleep(1) #waits for 1 second
```

# Ros2 Topics: publishers subscribers, to exchange messages

Topic:
- A named bus over which nodes exchange messages
- One topic can have multiple publishers - subscribers (each one of them is independant of the other)
- The message type must be specific, and it can be customized. The default "string" message with its "data" field can be used to start with

Practical Note:
as we have seen before, script name, node name, executable name are different. But sometimes, they could be the same.

To create a publisher inside a node
> node.create_publisher(String, "TopicName", 10)

10 is the default value of the queue size, which is the number of messages to keep in a buffer in case the application is late at handling the messages

To check available message type, the ros2 interface command can be used
> ros2 interface show <pakagename>/msg/<messagename>

the following string message can be used as a sample
> ros2 interface show example_interfaces/msg/String

the output is string data
type is string
field is data

to add this message type in our code we include a python import
> from example_interfaces.msg import String

we also add it as a package dependency, in the package.xml file
<depend>example_interfaces</depend>


```python
# we create a fucntion that will be used with this topic to publish
#this function will the the message type String that we included

def publishingfunction(self):

    #create a messge, of type string, and field data
    msg = String()
    msg.data = "Hello"
    self.publisher.publish(msg)
```

# Complete Publisher


```python
#!/usr/bin/env python3

#COMPLETE PUBLISHER

import rclpy
from rclpy.node import Node


#import the String message, from example_interfaces pakcage
from example_interfaces.msg import String


#create a class passing the Node class as argument
class MyNode(Node):

    #node initiation function
    def __init__(self):
        #consturcting the node, it will be come the "self" object of the class
        super().__init__("pyNode2")
        
        #create a publisher
        self.publisher_ = self.create_publisher(String, "InterestingTopic", 10)

        #create a timer to publish periodically
        self.timer = self.create_timer(1, self.pub)
        #output
        self.get_logger().info("Hello from this node")

        
    #create a function that handles the publishing
    def pub(self):
        #create a message
        msg = String()
        #fill the "data" field of this message
        msg.data = "Hi, this is a great message"

        #pulish the message with the publisher
        self.publisher_.publish(msg)
        self.get_logger().info("a new message has been published")
        self.get_logger().info(msg.data)
        #when you print , you print msg.data not msg
        #msg is the message object
        #msg.data is the message string field

def main():
    #start Ros communication
    rclpy.init()

    #define a node, give it a name (node constructor)
    node = MyNode()

    #keeps the ros communication spinning (keep your program running)
    rclpy.spin(node)
    
    #destroy node (optional, to be cleared in the backround or it will be done by garbage collector
    node.destroy_node()

    #shutdown ros communication
    rclpy.shutdown()


if __name__ == "__main__":
    main()
```

```python
# Subscriber node, create a subsciber node

#create a subsciber object
self.subscription = self.create_subscription(String,'topic',self.listener_callback,10)

#definde a callback function for the subsciber, this function, is called when a new message is recieved
#this funciton has self (inside the class) and msg The Ros message as argument
def listener_callback(self, msg):
    self.get_logger().info("This message was received")
    self.get_logger().info(msg.data)
```

# Complete Subscriber


```python
#!/usr/bin/env python3

#COMPLETE SUBSCIRBER

import rclpy
from rclpy.node import Node


#import the String message, from example_interfaces pakcage
from example_interfaces.msg import String


#create a class passing the Node class as argument
class MyNode(Node):

    #node initiation function
    def __init__(self):
        #consturcting the node, it will be come the "self" object of the class
        super().__init__("pyNode2")
        

        #create a subsciber object
        self.subscription = self.create_subscription(String,'InterestingTopic',self.listener_callback,10)

    #definde a callback function for the subsciber, this function, is called when a new message is recieved
    #this funciton has self (inside the class) and msg The Ros message as argument
    def listener_callback(self, msg):
        self.get_logger().info("This message was received")
        self.get_logger().info(msg.data)

    

def main():
    #start Ros communication
    rclpy.init()

    #define a node, give it a name (node constructor)
    node = MyNode()

    #keeps the ros communication spinning (keep your program running)
    try:
      rclpy.spin(node)
    
    #destroy node (optional, to be cleared in the backround or it will be done by garbage collector
    except key 
    node.destroy_node()

    #shutdown ros communication
    rclpy.shutdown()


if __name__ == "__main__":
    main()
```

# Debugging Topics
to check all available topics
> ros2 topic list

to print/simulate what is coming out of the topic
> ros2 topic echo /topicname

to check info on a topic
> ro2 topic info /topicname

/rosout is a hidden standard topic in ros2, when you log (print in ros2), messages are published on this topic
>/rosout

to show /rosout in rqt_graph you can remove debug and dead sinks markers

to check frequency rate of a topic in hz
> ros2 topic hz /topicname

to check the bandwidth (bits/sec)
> ros2 topic bw /topicname


# Rename a topic at runtime with remap

ros2 run <pakcagename> <executablename> --rs-args -r __node:newnodename -r oldtopicname:=newtopicname


# ROS2 Services
server, client
request, response

what are services

Services are another method of communication for nodes in the ROS graph. Services are based on a call-and-response model, versus topics’ publisher-subscriber model. While topics allow nodes to subscribe to data streams and get continual updates, services only provide data when they are specifically called by a client.

with topics we had to define the topic type (string etc.) it was called the interface a message (msg)

here we have to define the type for the request, and the response, (which could be different), the interface is called a Service

example_interfaces contain various service examples

ros2 interface show example_interfaces/srv/AddTwoInts

int64 a
int64 b
---
int64 sum

this will take two integers a and b
and retrun a third integer sum
a,b are part of the request
sum is part of the response


# Simple Server


```python
#Sample code for a simple server

import rclpy
from rclpy.node import Node

from example_interfaces.srv import AddTwoInts

class MinimalService(Node):

    def __init__(self):
        super().__init__('minimal_service')
        self.srv = self.create_service(AddTwoInts, 'add_two_ints', self.add_two_ints_callback)

    def add_two_ints_callback(self, request, response):
        response.sum = request.a + request.b
        self.get_logger().info("service received a request")

        return response


def main(args=None):
    rclpy.init(args=args)

    minimal_service = MinimalService()

    rclpy.spin(minimal_service)

    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

# in the below sys.args is used (optional)
#you can use it to pass arguments to python scripts, or ros nodes

before building
> python3 script.py arg1 arg2

after building
> ros2 run packagename nodename arg1 arg2


```python
#sample code for clinet

import sys

from example_interfaces.srv import AddTwoInts
import rclpy
from rclpy.node import Node


class MinimalClientAsync(Node):

    def __init__(self):
        super().__init__('minimal_client_async')
        self.cli = self.create_client(AddTwoInts, 'add_two_ints')
        while not self.cli.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('service not available, waiting again...')
        self.req = AddTwoInts.Request()

    def send_request(self, a, b):
        self.req.a = a
        self.req.b = b
        self.future = self.cli.call_async(self.req)
        rclpy.spin_until_future_complete(self, self.future)
        return self.future.result()


def main(args=None):
    rclpy.init(args=args)

    minimal_client = MinimalClientAsync()
    response = minimal_client.send_request(int(sys.argv[1]), int(sys.argv[2]))
    minimal_client.get_logger().info(
        'Result of add_two_ints: for %d + %d = %d' %
        (int(sys.argv[1]), int(sys.argv[2]), response.sum))

    minimal_client.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

# Services Debugging

to check available services
> ros2 service list

to simulate a clinet request to a service from the terminal, you can you the call function
> ros2 service call service_name service_type request_arguments

> ros2 service call /add_two_ints example_interfaces/srv/AddTwoInts "{a: 3, b: 4}"


# rosdep
(you might use is when working with third party packages)

ros tool used check and install dependencies (usually) before building

for example <depend>rclpy</depend>
if rclpy was not installed, rosdep can install it

install rosdep
> sudo apt install python3-rosdep2

initialize rosdep (For the first time)
> sudo rosdep init

> rosdep update

use rosdep
run the command inside your main workspace folder (src is the folder containing your code)

> rosdep install --from-paths src -y --ignore-src


# a word about git, github

git is a version control system, to manage code versions and repositories

installation
> sudo apt install git

git has many commands (not part of this lab) we will be just looking into the cloning feature:
for cloning (downloading) on anline repository

> git clone onlinelink

this will copy files, from online repo to your current folder

gihub.com (not to be confused with git) is a website (like many onther simlar sites), for hosting, online repositories

for example, ROS2 have their own github page

> https://github.com/ros2

containing several repositories, one of them is for examples

> https://github.com/ros2/examples

to clone it locally, go to a desired location and use git

> git clone https://github.com/ros2/exmaples


# Ros2  launch
for scaling up ros applications
used to launch multiple nodes at once

add a /launch folder to your package

create launch files in your launch folder


```python
# examplelaunch.py
#use the following script for python launch library, copy paste and add needed nodes from different packages 
import launch
import launch_ros.actions

def generate_launch_description():
    return launch.LaunchDescription([
        launch_ros.actions.Node(
            package='demo_nodes_cpp',
            executable='talker',
            name='talker'
        ),
        launch_ros.actions.Node(
            package='demo_nodes_cpp',
            executable='listener',
            name='listener'
        )
    ])
```

to launch in python pakcages, go to your launch folder and use ros2 launch
> ros2 launch examplelaunch.py


---
