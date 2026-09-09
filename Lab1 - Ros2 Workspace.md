
# WSL Setup Guide

---

## 🖥️ Install WSL
```bash
wsl --install --no-distribution
```

### In Case WSL could be installed because of virtulization setting, check below troubleshooting to enable it


---

## 📂 Create a New Folder
```bash
mkdir "C:\WSL\MyNewImage"
```

---

## 📦 Import the Tar Image  (Replace Path with your values)
```bash
wsl --import MyNewImage "FolderPath" "ImagePath"
```

**Example:**
```bash
wsl --import MyNewImage "C:\WSL\MyNewImage" "C:\Users\samer\mywsl.tar"
```

Check available images:
```bash
wsl --list
```



# Lab 1 — ROS 2 Basics

## ROS1 Distributions:
> [ROS1 Distributions](http://wiki.ros.org/Distributions)

## ROS2 Distributions:
> [ROS2 Releases](https://docs.ros.org/en/rolling/Releases.html)




### Examples of Ros nodes

```bash
ros2 run demo_nodes_cpp talker
```

```bash
ros2 run demo_nodes_py listener
```


### Running rqt graph node

```bash
rqt_graph
```


# ROS2 Packages

#### Creating a Workspace (a main folder to manage ROS packages)
- It consists of the folder and a `src` subfolder, which will contain the packages:
```bash
mkdir ~/ros2_ws/
```

#### Create a `src` Folder Inside the Workspace (used by the Colcon builder)
```bash
cd ~/ros2_ws/
mkdir src
```

#### Building the Whole Workspace
- Run this command in the workspace folder (it will create `install` folders containing executables):
```bash
colcon build
```

#### Sourcing the Workspace `setup.bash` to Make the Workspace Functionalities Available in Terminals
- Add this to your `.bashrc`:
```bash
source ~/ros2_ws/install/setup.bash
```

#### Creating a Python Package
1. Change directory to your `src` folder:
```bash
cd src
```

2. Use `ament` as the package creator, and `ament_python` for a Python package.
   - `colcon` will be the package builder.
   - Add dependencies (e.g., `rclpy`, the ROS client for Python to write ROS nodes in Python).

   **Note about Naming Packages:**
   - Package names should start with a lowercase letter and only contain lowercase letters, digits, underscores, and dashes.

3. Create the package:
```bash
ros2 pkg create --build-type ament_python <package_name> --dependencies rclpy
```

#### Building a Single Package in the Workspace
- To build a specific package, use the following command with `colcon`:
```bash
colcon build --packages-select <package_name>
```


# ROS2 Nodes

<img width="597" height="335" alt="image" src="https://github.com/user-attachments/assets/9b7f5cb9-bd34-40b2-ab9a-feb2eef647d3" />



# ROS2 Node Commands

Starting a node
```bash
ros2 run <package_name> <executable_name>
```

examples:
```bash
ros2 run turtlesim turtlesim_node
```

list running nodes:
```bash
ros2 node list
```

check node information
```bash
ros2 node info <node_name>
```

Check nodes on RQT graph
```bash
rqt_graph
```


for example
```bash
pip3 install setuptools==58
```


# Raw Ros2 node

not structured as a class


```bash
#raw ros2 python node (without a main funtion, or a class sturcture)
#this is the most basic form of using rclpy library

#importing ros2 clinet library for python
import rclpy
#also importing the Node class inside it (more practical)
from rclpy.node import Node

#start Ros communication
rclpy.init()

#define a node, give it a name (node constructor)
node = Node("pyNode1")

#output something with the node
node.get_logger().info("hello node")

#keeps the node spinning (keep your program running)
#rclpy.spin(node)


#shutdown ros communication
rclpy.shutdown()
```


# Complete Ros2 Node structured as a class


```python
#!/usr/bin/env python3

#RO2 node, written in python, written as a class consturcted with "Node" class from rclpy, and called from an external function that can be built.
#this is the recommended way or writing a ros2 node in python (scalable, modular, and most documentations online use this format)


import rclpy
from rclpy.node import Node


#create a class passing the Node class as argument
class MyNode(Node):

    #node initiation function
    def __init__(self):
        #consturcting the node, it will be come the "self" object of the class
        super().__init__("pyNode2")
        
        #output
        self.get_logger().info("Hello from new node 2")



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

```bash
# modify the setup.py file of the package to install/build the ros2 node (creates executables)

entry_points={
    'console_scripts': [
        "PyNode1exe = PakcageName.ScriptName:functionname",
        "PyNode2exe = PakcageName.ScriptName:functionname"
    ],
},
```



# colcon notes

```bash
colcon build --packages-select packagename --symlink-install
```

(works only for python)
build the package in a way, that each time the code is changed and the executable is triggered, you don't need to rebuild (this is efficient when developing - for example when changing the name of the node)

also to work the python file must be an executable (allow to execute the python file as an OS program)
```bash
chmod +x pythonscriptname.py
```

when listing ls in terminal executables are colored in green
you can also right click the file, properties, permissions, and set it to execute


---


## Lab 3 — Publishers, Subscribers, Services

# Timers, Calback Functions


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
