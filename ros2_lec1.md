# 🤖 ROS2 Humble — Lecture 1
### Complete Beginner Notes | Naval Architecture & Marine Engineering, BUET

---

## 📋 Table of Contents

1. [What is ROS2?](#1-what-is-ros2)
2. [Key Concepts](#2-key-concepts)
3. [Installation Steps](#3-installation-steps)
4. [Sourcing ROS2](#4-sourcing-ros2)
5. [Workspace Setup](#5-workspace-setup)
6. [Creating a Package](#6-creating-a-package)
7. [setup.py — Entry Points](#7-setuppy--entry-points)
8. [Creating Python Node Files](#8-creating-python-node-files)
9. [General Node Structure](#9-general-node-structure-python)
10. [Publisher Node](#10-publisher-node)
11. [Subscriber Node](#11-subscriber-node)
12. [Publisher vs Subscriber](#12-publisher-vs-subscriber)
13. [Build and Run Workflow](#13-build-and-run-workflow)
14. [Running a Node as a Python Script](#14-running-a-node-as-a-python-script)
15. [Debugging Commands](#15-debugging-commands)
16. [Terminal Command Reference](#16-terminal-command-reference)

---

## 1. What is ROS2?

**ROS2** (Robot Operating System 2) is a **framework** for building robot software. It is NOT a real operating system — it is a collection of tools, libraries, and conventions that lets different parts of a robot communicate with each other using a **message-passing system**.

> **Platform Requirement:** ROS2 Humble requires **Ubuntu 22.04 LTS**. No other Ubuntu version works cleanly.

### 🎙️ Radio Analogy

| ROS2 Concept | Radio Analogy |
|---|---|
| **Topic** | Radio frequency (e.g. 101.5 FM) |
| **Publisher** | Radio station broadcasting on that frequency |
| **Subscriber** | Your radio receiver tuned to that frequency |

---

## 2. Key Concepts

| Concept | What it means |
|---|---|
| **Node** | A single program/process. A robot runs many nodes (camera node, motor node, etc.) |
| **Topic** | A named channel that nodes use to send/receive data |
| **Publisher** | A node that sends messages onto a topic |
| **Subscriber** | A node that listens to a topic and receives messages automatically |
| **Message** | The data being sent — can be a string, number, sensor reading, etc. |
| **Package** | A folder that groups related nodes and code together |
| **Workspace** | Your working directory where you write, build, and run ROS2 code |

---

## 3. Installation Steps

### Step 1 — Check Ubuntu Version
```bash
lsb_release -a
```
> Must show **Ubuntu 22.04**

### Step 2 — Set Locale
```bash
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

### Step 3 — Enable Universe Repository
```bash
sudo apt install software-properties-common
sudo add-apt-repository universe
```

### Step 4 — Add ROS2 GPG Key
```bash
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg
```

### Step 5 — Add ROS2 Repository
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
  http://packages.ros.org/ros2/ubuntu \
  $(. /etc/os-release && echo $UBUNTU_CODENAME) main" \
  | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

### Step 6 — Install ROS2 Humble Desktop
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install ros-humble-desktop -y
```
> ⏳ This downloads **300–600 MB**. Takes several minutes.

### Step 7 — Install Dev Tools
```bash
sudo apt install python3-colcon-common-extensions -y
sudo apt install python3-rosdep -y
sudo apt install python3-argcomplete -y
```

---

## 4. Sourcing ROS2

**"Sourcing"** tells your terminal where ROS2 is installed by setting environment variables like `PATH`, `PYTHONPATH`, `AMENT_PREFIX_PATH`, etc.

> Without sourcing, your terminal has **no idea** where ROS2 is and commands like `ros2` won't work.

### Once (current terminal only)
```bash
source /opt/ros/humble/setup.bash
```

### Permanently (auto-sources every new terminal)
```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### Verify installation
```bash
ros2 --help
```

### Quick sanity test — run the built-in demo

**Terminal 1:**
```bash
ros2 run demo_nodes_cpp talker
```

**Terminal 2:**
```bash
ros2 run demo_nodes_py listener
```
You should see `Hello World: 1`, `Hello World: 2`... flowing between them. That's a publisher and subscriber in action! Kill both with `Ctrl+C`.

---

## 5. Workspace Setup

A **workspace** is just a folder where you write and build your ROS2 packages.

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
colcon build
```

Then source your workspace permanently:
```bash
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### Workspace folder structure

```
ros2_ws/
├── src/        ← your packages go here (YOU write code here)
├── build/      ← auto-generated during build
├── install/    ← compiled code lives here
└── log/        ← build logs
```

---

## 6. Creating a Package

```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python my_first_pkg
```

### Auto-generated package structure

```
my_first_pkg/
├── my_first_pkg/
│   └── __init__.py     ← marks it as a Python module
├── package.xml         ← metadata: name, version, dependencies
├── resource/
│   └── my_first_pkg
├── setup.cfg           ← config for the build system
└── setup.py            ← where you register nodes as entry points
```

---

## 7. setup.py — Entry Points

`setup.py` tells ROS2 **how to run your nodes**. The critical section is `entry_points`:

```python
entry_points={
    'console_scripts': [
        'publisher_node  = my_first_pkg.publisher_node:main',
        'subscriber_node = my_first_pkg.subscriber_node:main',
    ],
},
```

### Format breakdown

```
'EXECUTABLE_NAME = PACKAGE_NAME.FILE_NAME:FUNCTION_NAME'
```

| Part | Value | Meaning |
|---|---|---|
| `publisher_node` | executable name | what you type after `ros2 run my_first_pkg` |
| `my_first_pkg` | package name | the folder under `src/` |
| `publisher_node` | Python file name | `publisher_node.py` (no `.py` here) |
| `main` | function name | the `def main():` inside that file |

> ⚠️ **Every time you add a new node file**, add a new line to `entry_points`, then rebuild with `colcon build`.

---

## 8. Creating Python Node Files

Before you write any node code, you need to **create the `.py` file** inside your package folder. There are several ways to do this.

### Where to create the file

Always create node files inside the **inner package folder** (same name as your package):

```
ros2_ws/
└── src/
    └── my_first_pkg/
        └── my_first_pkg/       ← CREATE YOUR .py FILES HERE
            ├── __init__.py
            ├── publisher_node.py
            └── subscriber_node.py
```

```bash
# Navigate there first
cd ~/ros2_ws/src/my_first_pkg/my_first_pkg
```

---

### Method 1 — `nano` (terminal text editor, recommended for beginners)

```bash
nano publisher_node.py
```

This opens the nano editor right in your terminal. Write your code, then:

| Key | Action |
|---|---|
| `Ctrl + O` | Save the file (WriteOut) |
| `Enter` | Confirm the filename |
| `Ctrl + X` | Exit nano |

> `nano` is the simplest terminal editor — no modes, no commands to memorize. What you type is what you get.

---

### Method 2 — `touch` then open with any editor

`touch` creates an **empty file** without opening it:

```bash
touch publisher_node.py
```

Then open it with whatever editor you prefer:

```bash
nano publisher_node.py      # terminal editor (simplest)
gedit publisher_node.py     # GUI text editor (like Notepad)
code publisher_node.py      # VS Code (if installed)
```

> `touch` is useful when you want to create the file first and open it later, or create multiple files at once.

---

### Method 3 — `gedit` (GUI editor, easiest to use)

```bash
gedit publisher_node.py &
```

Opens a graphical text editor window. The `&` at the end lets it run in the background so your terminal stays usable.

> Best if you're not comfortable with terminal editors yet.

---

### Method 4 — VS Code (best for larger projects)

If VS Code is installed:

```bash
code publisher_node.py      # open one file
code .                      # open entire workspace folder in VS Code
```

> Recommended once you're past the basics — syntax highlighting, autocomplete, and integrated terminal make development much faster.

---

### Quick Comparison

| Method | Command | Best for |
|---|---|---|
| `nano` | `nano file.py` | Quick edits in terminal, no GUI needed |
| `touch` + editor | `touch file.py` then open | Creating file first, editing later |
| `gedit` | `gedit file.py &` | Comfortable GUI editing |
| VS Code | `code file.py` | Full development experience |

---

### ⚠️ Common Mistake — Wrong Folder

A very common beginner mistake is creating the file in the **wrong folder**:

```
my_first_pkg/
├── my_first_pkg/        ← ✅ CORRECT — create .py files HERE
│   ├── __init__.py
│   └── publisher_node.py
├── package.xml
└── setup.py             ← ❌ WRONG — do NOT put .py files next to setup.py
```

If you put your node file in the outer folder (next to `setup.py`), `ros2 run` will never find it.

---

## 9. General Node Structure (Python)

```python
import rclpy                          # main ROS2 Python library
from rclpy.node import Node           # base class for ALL nodes
from std_msgs.msg import String       # message type (string)

class MyNode(Node):                   # inherit from Node

    def __init__(self):
        super().__init__('node_name') # ROS2 name for this node
        # create publishers, subscribers, timers here ...

def main(args=None):
    rclpy.init(args=args)    # initialize ROS2 — ALWAYS FIRST
    node = MyNode()           # create your node object
    rclpy.spin(node)          # keep it alive (event loop — waits for callbacks)
    node.destroy_node()       # clean shutdown
    rclpy.shutdown()          # shutdown ROS2 — ALWAYS LAST

if __name__ == '__main__':   # runs only when executed as script directly
    main()
```

### Line-by-line explanation

| Line | What it does |
|---|---|
| `import rclpy` | Loads the ROS2 Python library — every node needs this |
| `from rclpy.node import Node` | Imports the base class your node will inherit from |
| `class MyNode(Node)` | Your node is a Python class that inherits Node's superpowers |
| `super().__init__('node_name')` | Registers your node with ROS2 under that name |
| `rclpy.init()` | Starts the ROS2 system — must be called before anything else |
| `rclpy.spin(node)` | Event loop — keeps node alive, fires callbacks when needed |
| `node.destroy_node()` | Cleans up the node after spin exits (Ctrl+C) |
| `rclpy.shutdown()` | Shuts down the ROS2 system — must be last |

---

## 10. Publisher Node

A publisher **sends messages** onto a topic at a fixed interval using a timer.

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MyPublisher(Node):

    def __init__(self):
        super().__init__('my_publisher')

        # Create publisher: (message_type, topic_name, queue_size)
        self.publisher_ = self.create_publisher(String, 'my_topic', 10)

        # Create timer: fires every 1.0 seconds → calls timer_callback
        self.timer = self.create_timer(1.0, self.timer_callback)
        self.count = 0
        self.get_logger().info('Publisher node has started!')

    def timer_callback(self):
        msg = String()                                  # create empty message
        msg.data = f'Hello ROS2! Count: {self.count}'  # fill in data
        self.publisher_.publish(msg)                    # send it out
        self.get_logger().info(f'Publishing: "{msg.data}"')
        self.count += 1

def main(args=None):
    rclpy.init(args=args)
    node = MyPublisher()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### Publisher flow

```
Timer fires every 1s
      │
      ▼
timer_callback()
      │
      ▼
msg = String()          ← create message
msg.data = "Hello..."   ← fill data
      │
      ▼
publisher_.publish(msg) ← send to /my_topic
```

### `create_publisher` arguments

| Argument | Value | Meaning |
|---|---|---|
| Message type | `String` | What kind of data to send |
| Topic name | `'my_topic'` | The channel name (must match subscriber) |
| Queue size | `10` | Buffer size — how many messages to hold if sending faster than network |

---

## 11. Subscriber Node

A subscriber **listens to a topic** and automatically calls a callback function when a message arrives.

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MySubscriber(Node):

    def __init__(self):
        super().__init__('my_subscriber')

        # Create subscriber: (message_type, topic_name, callback, queue_size)
        self.subscription = self.create_subscription(
            String,
            'my_topic',              # MUST match publisher's topic name
            self.listener_callback,  # function called when message arrives
            10
        )
        self.get_logger().info('Subscriber started, waiting for messages...')

    def listener_callback(self, msg):
        # Called AUTOMATICALLY by ROS2 every time a message arrives
        self.get_logger().info(f'I heard: "{msg.data}"')

def main(args=None):
    rclpy.init(args=args)
    node = MySubscriber()
    rclpy.spin(node)        # waits here, fires callback on each message
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### Subscriber flow

```
Message arrives on /my_topic
            │
            ▼
  ROS2 detects new message
            │
            ▼
  listener_callback(msg) called automatically
            │
            ▼
  msg.data → read and use the data
```

---

## 12. Publisher vs Subscriber

| Aspect | Publisher | Subscriber |
|---|---|---|
| **Creates with** | `create_publisher(Type, topic, qsize)` | `create_subscription(Type, topic, callback, qsize)` |
| **Sends/receives** | Calls `.publish(msg)` manually | Callback fired **automatically** by ROS2 |
| **Needs a timer?** | Usually yes — to send at intervals | No — callback fires on message arrival |
| **Topic name** | Defines the topic | Must **match** the publisher's topic name |
| **Callback role** | Creates and sends a message | Receives and reads a message |

### How they connect

```
[Publisher Node]                              [Subscriber Node]
      │                                              │
create_publisher(String, 'my_topic', 10)     create_subscription(String, 'my_topic', cb, 10)
      │                                              │
      │         ROS2 Middleware                      │
      │    ┌─────────────────────┐                   │
      └───▶│      /my_topic      │──────────────────▶│
           │   (String messages) │            listener_callback(msg)
           └─────────────────────┘
```

> The two nodes **never directly talk to each other**. They only know about the **topic name**.
> This means you can have multiple publishers and subscribers on one topic, even across different machines.

---

## 13. Build and Run Workflow

### Every time you write or edit a node:

```bash
# Step 1 — Build
cd ~/ros2_ws
colcon build --packages-select my_first_pkg

# Step 2 — Re-source (MUST do this after every build)
source ~/.bashrc

# Step 3 — Run publisher (Terminal 1)
ros2 run my_first_pkg publisher_node

# Step 4 — Run subscriber (Terminal 2)
ros2 run my_first_pkg subscriber_node
```

> ⚠️ **You MUST re-source after every build.** This is the most common beginner mistake.

---

## 14. Running a Node as a Python Script

You can skip the build step and run directly as a Python script — useful during development:

```bash
python3 ~/ros2_ws/src/my_first_pkg/my_first_pkg/publisher_node.py
```

Or as a module:
```bash
cd ~/ros2_ws/src/my_first_pkg
python3 -m my_first_pkg.publisher_node
```

### Comparison

| Method | Needs `colcon build`? | Needs `setup.py` entry? | Best for |
|---|---|---|---|
| `ros2 run` | ✅ Yes | ✅ Yes | Production / launch files |
| `python3 script.py` | ❌ No | ❌ No | Quick testing during development |
| `python3 -m` | ❌ No | ❌ No | Module-style testing |

> All three methods work the same way with ROS2 — `ros2 topic echo` and `ros2 node list` will see the node regardless of how it was started.

---

## 15. Debugging Commands

Run these in a **separate terminal** while your nodes are running:

```bash
# List all currently running nodes
ros2 node list

# List all active topics
ros2 topic list

# Print messages flowing on a topic in real time (like wiretapping)
ros2 topic echo /my_topic

# Show publish rate of a topic (messages per second)
ros2 topic hz /my_topic

# Show message type, publishers, subscribers of a topic
ros2 topic info /my_topic

# Show what topics a node publishes/subscribes to
ros2 node info /my_publisher
```

---

## 16. Terminal Command Reference

### Package Manager (`apt`)

| Command | What it does |
|---|---|
| `sudo apt update` | Refresh the package list — does **NOT** install anything |
| `sudo apt upgrade -y` | Upgrade all installed packages to latest versions |
| `sudo apt install X -y` | Install package X (`-y` skips confirmation prompts) |
| `sudo add-apt-repository universe` | Enable the community-maintained software repo |

### Shell & File Commands

| Command | What it does |
|---|---|
| `lsb_release -a` | Show Ubuntu version info |
| `source file.bash` | Run a shell script in current terminal (sets env variables) |
| `echo "text" >> file` | **Append** text to end of file (`>>` appends, `>` overwrites — careful!) |
| `source ~/.bashrc` | Reload `.bashrc` in current terminal immediately |
| `mkdir -p path/to/folder` | Create folder and all parent folders in one shot |
| `touch file.py` | Create an empty file (does not open it) |
| `nano file.py` | Create/open a file in the nano terminal text editor |
| `gedit file.py &` | Open a file in the GUI text editor (`&` keeps terminal free) |
| `code file.py` | Open a file in VS Code |
| `code .` | Open the entire current folder in VS Code |

### ROS2 Commands

| Command | What it does |
|---|---|
| `colcon build` | Build all packages in the workspace |
| `colcon build --packages-select X` | Build only package X (faster during development) |
| `ros2 pkg create --build-type ament_python X` | Create a new Python ROS2 package named X |
| `ros2 run <pkg> <node>` | Run a node from a package |
| `ros2 node list` | List all currently running nodes |
| `ros2 topic list` | List all active topics |
| `ros2 topic echo /topic` | Print messages on a topic in real time |
| `ros2 topic hz /topic` | Show publish rate of a topic |
| `ros2 topic info /topic` | Show type, publishers, subscribers of a topic |
| `ros2 node info /node` | Show what a node publishes/subscribes to |

---

## 🗺️ Everything at a Glance

```
Write code in  →  ~/ros2_ws/src/my_first_pkg/my_first_pkg/*.py
Register in    →  setup.py → entry_points
Build with     →  colcon build --packages-select my_first_pkg
Source with    →  source ~/.bashrc
Run with       →  ros2 run my_first_pkg <node_name>
           or  →  python3 your_node.py  (for quick testing)
Debug with     →  ros2 topic echo / ros2 node list / ros2 topic hz
```

---

## ➡️ What's Next

- **Custom Message Types** — define your own message structure beyond `std_msgs`
- **Services** — request/response communication (instead of publish/subscribe)
- **Launch Files** — start multiple nodes at once with a single command
- **Parameters** — configure nodes at runtime without changing code
- **TF2** — coordinate frame transformations for robotics

---

*Shozib | Naval Architecture & Marine Engineering, BUET*
