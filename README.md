# Robotics Software Engineer:
### A 9-Week Hands-On Learning Roadmap
**Target Date:** August 25, 2026 | **Pace:** ~15 hrs/week | **Focus:** ROS2, Gazebo, Nav2, MoveIt2, OpenRMF

---

## Feasibility Assessment

**July 31 (5.5 weeks / ~82 hrs):** Not enough. Covering ROS2 fundamentals alone takes ~25–30 hours done properly. You'd reach Nav2 competency but have almost no time for MoveIt2 or OpenRMF. Rushing these would leave you unable to answer interview questions confidently.

**August 25 (9 weeks / ~135 hrs):** The right target. You'll have meaningful hands-on experience across all four frameworks, 4–5 GitHub projects to show, and enough depth to handle entry/junior robotics software engineering interviews. "Expert" is a high bar for any 9-week timeline — think of this as getting you to **job-ready intermediate** level, which is exactly what "hireable" means in this space.

**Assumption:** You have basic Python comfort and a GitHub account. If you're also comfortable with C++, even better — but Python is enough to start.

---

## Environment Setup (Do This First — ~3 hrs, before Week 1)

Everything you'll work with runs on **Ubuntu 22.04** (ROS2 Humble) or **Ubuntu 24.04** (ROS2 Jazzy). You have two options:

**Option A — Native Ubuntu (recommended):** Dual-boot or dedicated machine. Best performance for simulation.

**Option B — Docker (quickest start):** Run everything in containers, no OS changes needed.

```bash
# Docker quick-start (run on any OS)
docker pull osrf/ros:humble-desktop
docker run -it --rm osrf/ros:humble-desktop bash
```

For GUI apps (Gazebo, RViz), use [Rocker](https://github.com/osrf/rocker) or [VcXsrv](https://sourceforge.net/projects/vcxsrv/) on Windows.

**Cloud alternative:** [The Construct](https://www.theconstructsim.com) gives you a full ROS2 + Gazebo environment in the browser. Great for when you can't run locally. The free tier is limited but usable for learning.

**Install ROS2 Humble (Ubuntu 22.04):**
```bash
sudo apt update && sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
  http://packages.ros.org/ros2/ubuntu \
  $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | \
  sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
sudo apt update
sudo apt install ros-humble-desktop python3-colcon-common-extensions
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

**GitHub repo to create now:** Make a repo called `robotics-learning-journey`. This will be your umbrella — each week's exercises go in here as sub-folders or branches.

---

## Phase 1: ROS2 Foundations
### Weeks 1–2 | ~30 hours | GitHub project: `ros2-fundamentals`

ROS2 (Robot Operating System 2) is the backbone of everything else you'll learn. Nav2, MoveIt2, and OpenRMF all run on top of it. Get this solid before moving on.

---

### Week 1 — Core Concepts (15 hrs)

**Concepts to learn:**
- What ROS2 is and why it replaced ROS1 (DDS middleware, real-time capability, no rosmaster)
- **Nodes**: individual processes that do one thing (e.g., read a sensor, control a wheel)
- **Topics**: named data streams; nodes publish to or subscribe from them
- **Messages**: typed data packets sent over topics (e.g., `geometry_msgs/Twist` for velocity)
- **ROS2 graph**: the network of nodes and topics at runtime
- **Workspaces and packages**: how ROS2 organizes code
- **colcon**: the build tool (like `make` but for ROS2 packages)

**Practice:**
```bash
# After installing ROS2, verify it works
ros2 run demo_nodes_py talker   # in terminal 1
ros2 run demo_nodes_py listener # in terminal 2
ros2 topic list                 # see active topics
ros2 topic echo /chatter        # inspect messages
```

**Build your first package:**
```bash
mkdir -p ~/ros2_ws/src && cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python my_first_pkg --dependencies rclpy std_msgs
cd ~/ros2_ws
colcon build
source install/setup.bash
```

**Write a publisher node** (`my_publisher.py`):
```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MyPublisher(Node):
    def __init__(self):
        super().__init__('my_publisher')
        self.publisher_ = self.create_publisher(String, 'my_topic', 10)
        self.timer = self.create_timer(1.0, self.timer_callback)

    def timer_callback(self):
        msg = String()
        msg.data = f'Hello from ROS2! Count: {self.get_clock().now()}'
        self.publisher_.publish(msg)
        self.get_logger().info(f'Publishing: {msg.data}')

def main():
    rclpy.init()
    node = MyPublisher()
    rclpy.spin(node)
    rclpy.shutdown()
```

**Write a subscriber node** and make them talk to each other. Push both to GitHub.

**Resources for Week 1:**
- [ROS2 Official Tutorials — Beginner: CLI Tools](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools.html) — do ALL of them
- [Robotics Backend YouTube: ROS2 for Beginners playlist](https://www.youtube.com/@RoboticsBackEnd) — the most practical free resource available
- [ROS2 for Beginners — Udemy by Edouard Renard](https://www.udemy.com/course/ros2-for-beginners/) (~$15 on sale) — highly recommended if you want structured video content

**By end of Week 1 you should be able to:** Create a ROS2 package, write publisher/subscriber nodes, inspect the ROS2 graph with `ros2 topic`, and build with colcon.

---

### Week 2 — Services, Actions, TF2, Launch Files (15 hrs)

**Concepts to learn:**
- **Services**: request/response pattern (synchronous); unlike topics which are continuous
- **Actions**: long-running tasks with feedback and cancel support (e.g., "navigate to goal")
- **Parameters**: runtime configuration values for nodes
- **Launch files**: Python files that start multiple nodes at once with configuration
- **TF2**: the transform library — tracks coordinate frames (robot base, camera, map, etc.) over time
- **URDF**: Unified Robot Description Format — XML that describes a robot's geometry and joints

**Practice — write a Service:**
```python
# Server
from std_srvs.srv import AddTwoInts

class AdditionServer(Node):
    def __init__(self):
        super().__init__('addition_server')
        self.srv = self.create_service(AddTwoInts, 'add_two_ints', self.add_callback)

    def add_callback(self, request, response):
        response.sum = request.a + request.b
        return response
```

**Practice — write an Action:**
```bash
# Create a custom action that counts to N and reports progress
# Action type: Fibonacci sequence generator (classic ROS2 example)
ros2 action list   # see active actions
ros2 action info /fibonacci
```

**Write a launch file:**
```python
# my_launch.py
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(package='my_pkg', executable='my_publisher', name='pub1'),
        Node(package='my_pkg', executable='my_subscriber', name='sub1'),
    ])
```

**Resources for Week 2:**
- [ROS2 Official Tutorials — Beginner: Client Libraries](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries.html)
- [TF2 Tutorials](https://docs.ros.org/en/humble/Tutorials/Intermediate/Tf2/Tf2-Main.html)
- [URDF Tutorials](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/URDF-Main.html)

**GitHub commit at end of Week 2:** Your `ros2-fundamentals` folder should have a publisher, subscriber, service server/client, action server/client, and a launch file. Write a proper README explaining what each does.

---

## Phase 2: Gazebo Simulation
### Week 3 | ~15 hours | GitHub project: `custom-robot-gazebo`

Gazebo is the standard robot simulator in ROS2. You'll use it for everything else — Nav2 and MoveIt2 both test in Gazebo.

**Which Gazebo?** Use **Gazebo Harmonic** (also called Ignition Gazebo or "New Gazebo"). NOT Gazebo Classic (deprecated). With ROS2 Humble, you want Gazebo Fortress or Harmonic + the `ros_gz` bridge.

```bash
sudo apt install ros-humble-ros-gz
```

**Concepts to learn:**
- SDF (Simulation Description Format): Gazebo's world/model format
- Spawning robot models from URDF in Gazebo
- `ros_gz_bridge`: passes data between Gazebo topics and ROS2 topics
- Sensor plugins: how to add lidar, camera, IMU to a simulated robot
- World files: how to create environments (rooms, obstacles, etc.)
- Gazebo GUI: pause/play, inserting models, visual debugging

**Practice — build a differential drive robot:**

Step 1: Create a URDF with:
- A base link (box or cylinder)
- Two wheels (connected with continuous joints)
- A caster wheel (for balance)
- A lidar sensor on top

Step 2: Spawn it in Gazebo and control it:
```bash
# Bridge velocity commands from ROS2 to Gazebo
ros2 run ros_gz_bridge parameter_bridge \
  /cmd_vel@geometry_msgs/msg/Twist@gz.msgs.Twist

# Drive it with keyboard
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

Step 3: Visualize in RViz2:
```bash
rviz2  # add RobotModel and LaserScan displays
```

Step 4: Build a simple world with walls and obstacles.

**Resources for Week 3:**
- [Gazebo Tutorials (official)](https://gazebosim.org/docs)
- [Articulated Robotics YouTube — URDF + Gazebo series](https://www.youtube.com/@ArticulatedRobotics) — the single best tutorial series for this phase
- [ros_gz GitHub repo](https://github.com/gazebosim/ros_gz) — reference for bridge topics

**GitHub commit:** A repo with your robot URDF, a Gazebo world file, a launch file that starts everything, and screenshots/GIFs of the robot moving. This is already impressive for a portfolio piece.

---

## Phase 3: Navigation (Nav2)
### Weeks 4–5 | ~30 hours | GitHub project: `nav2-autonomous-explorer`

Nav2 is the ROS2 navigation stack. It lets a robot perceive its environment, build a map, localize itself, and plan paths to goals. This is one of the most in-demand skills in robotics software.

---

### Week 4 — Nav2 Concepts + SLAM (15 hrs)

**Concepts to learn:**
- **Costmaps**: layered occupancy grids representing traversable vs. obstacle space
  - Global costmap: used for long-range path planning
  - Local costmap: used for immediate obstacle avoidance
- **Global planner**: finds a path from A to B on the map (e.g., NavFn, Smac Planner)
- **Local controller/follower**: executes the path in real time (e.g., DWB, RPP, MPPI)
- **SLAM**: Simultaneous Localization and Mapping — builds a map while navigating
- **AMCL**: Adaptive Monte Carlo Localization — localizes on an existing map using particles
- **Lifecycle nodes**: Nav2 uses managed nodes that go through configure → activate → deactivate states
- **Behavior Trees**: Nav2 orchestrates its actions through BehaviorTree.CPP trees

**Install Nav2:**
```bash
sudo apt install ros-humble-nav2-bringup ros-humble-slam-toolbox
```

**Run the Nav2 demo:**
```bash
# Launch TurtleBot3 in Gazebo (easiest way to learn Nav2)
sudo apt install ros-humble-turtlebot3-gazebo
export TURTLEBOT3_MODEL=burger
ros2 launch nav2_bringup tb3_simulation_launch.py
```

This gives you a robot in a simulated house. Use RViz2 to set a "Nav2 Goal" and watch it navigate.

**Then set up SLAM:**
```bash
ros2 launch slam_toolbox online_async_launch.py
# Drive the robot manually with teleop — watch the map build in RViz2
```

**Resources for Week 4:**
- [Nav2 Official Documentation](https://navigation.ros.org) — bookmark this and read the Concepts section
- [Nav2 Getting Started tutorial](https://navigation.ros.org/getting_started/index.html)
- [Robotics Backend — Nav2 tutorials](https://www.youtube.com/@RoboticsBackEnd)

---

### Week 5 — Nav2 Advanced + Custom Configuration (15 hrs)

**Concepts to learn:**
- Writing custom behavior tree nodes
- Nav2 parameter tuning (costmap inflation radius, controller frequency, planner tolerance)
- Waypoint following: send the robot through a sequence of goals
- Recovery behaviors: what happens when the robot gets stuck
- Costmap plugins: static layer, obstacle layer, inflation layer, voxel layer
- Nav2 with your own robot (not TurtleBot3)

**Practice — build your own Nav2 config:**

1. Take the custom Gazebo robot from Week 3
2. Add Nav2 to it with a custom parameter YAML:
```yaml
# nav2_params.yaml (excerpt)
controller_server:
  ros__parameters:
    controller_frequency: 20.0
    FollowPath:
      plugin: "nav2_regulated_pure_pursuit_controller::RegulatedPurePursuitController"
      desired_linear_vel: 0.3
      max_angular_accel: 2.0
```
3. Map an environment with slam_toolbox, save the map
4. Switch to localization-only mode with AMCL + saved map
5. Send autonomous waypoint goals via the Nav2 Python API:

```python
from nav2_simple_commander.robot_navigator import BasicNavigator
from geometry_msgs.msg import PoseStamped

navigator = BasicNavigator()
goal = PoseStamped()
goal.header.frame_id = 'map'
goal.pose.position.x = 2.0
goal.pose.position.y = 1.5
navigator.goToPose(goal)

while not navigator.isTaskComplete():
    feedback = navigator.getFeedback()
    print(f'Distance remaining: {feedback.distance_remaining:.2f}m')
```

**GitHub project for Weeks 4–5:** A complete autonomous navigation demo — robot spawns in Gazebo, maps the environment, saves the map, then navigates to a series of waypoints autonomously. Include a README with a GIF, the map image, and instructions to reproduce. This is a strong portfolio project.

**Resources for Week 5:**
- [Nav2 Simple Commander API](https://navigation.ros.org/commander_api/index.html)
- [BehaviorTree.CPP documentation](https://www.behaviortree.dev)
- [Nav2 tuning guide](https://navigation.ros.org/tuning/index.html)

---

## Phase 4: Manipulation (MoveIt2)
### Weeks 6–7 | ~30 hours | GitHub project: `moveit2-pick-place`

MoveIt2 handles robot arm manipulation — planning and executing motion for arms with multiple joints. It's the standard framework used in industrial and research robotics.

---

### Week 6 — MoveIt2 Fundamentals (15 hrs)

**Concepts to learn:**
- **Kinematics**: Forward Kinematics (joint angles → end-effector pose), Inverse Kinematics (target pose → joint angles)
- **Planning scene**: virtual representation of the robot and environment (what's in the way)
- **MoveGroup**: the main MoveIt2 interface for planning and executing motion
- **SRDF**: Semantic Robot Description Format — extends URDF with planning groups, end effectors
- **MoveIt Setup Assistant**: GUI that auto-generates MoveIt config for any URDF
- **OMPL**: Open Motion Planning Library — the default motion planning algorithms in MoveIt2
- **ros2_control**: the hardware interface layer MoveIt2 talks to

**Install MoveIt2:**
```bash
sudo apt install ros-humble-moveit ros-humble-moveit-setup-assistant
```

**Start with Panda arm (the tutorial robot):**
```bash
ros2 launch moveit2_tutorials demo.launch.py
```

This gives you the Panda robot arm in RViz2 with a motion planning panel. Plan and execute moves by dragging the end-effector to a target pose.

**Run the MoveIt Setup Assistant on your own robot:**
```bash
ros2 launch moveit_setup_assistant setup_assistant.launch.py
# Load your URDF → configure joints → auto-generate config package
```

**Write your first MoveGroup Python node:**
```python
from moveit.planning import MoveItPy

moveit = MoveItPy(node_name="moveit_py")
arm = moveit.get_planning_component("panda_arm")

# Plan to a named target
arm.set_start_state_to_current_state()
arm.set_goal_state(configuration_name="ready")
plan_result = arm.plan()

if plan_result:
    moveit.execute(plan_result.trajectory, blocking=True)
```

**Resources for Week 6:**
- [MoveIt2 Documentation](https://moveit.picknik.ai/main/index.html)
- [MoveIt2 Getting Started](https://moveit.picknik.ai/main/doc/tutorials/getting_started/getting_started.html)
- [Robotics Backend — MoveIt2 tutorials](https://www.youtube.com/@RoboticsBackEnd)
- [MoveIt2 Python tutorials](https://moveit.picknik.ai/main/doc/examples/moveit_py/moveit_py_tutorial.html)

---

### Week 7 — MoveIt2 Advanced + Pick and Place (15 hrs)

**Concepts to learn:**
- **Collision objects**: adding boxes, cylinders, meshes to the planning scene
- **Attached objects**: object attaches to the gripper after grasp
- **Cartesian path planning**: moving the end-effector in a straight line
- **MoveIt Task Constructor (MTC)**: a task-level planning framework built on top of MoveIt — defines complex manipulation tasks as stages
- **Gripper control**: using ros2_control to open/close a gripper
- **Grasp generation**: computing grasp poses for objects

**Practice — pick and place pipeline:**

```python
# Add a collision object (the box to pick)
from moveit_msgs.msg import CollisionObject
from shape_msgs.msg import SolidPrimitive

collision_object = CollisionObject()
collision_object.id = "box"
collision_object.header.frame_id = "world"
box = SolidPrimitive()
box.type = SolidPrimitive.BOX
box.dimensions = [0.05, 0.05, 0.05]
collision_object.primitives.append(box)
# ... set pose and add to planning scene

# Cartesian path to pre-grasp pose
waypoints = []
waypoints.append(pre_grasp_pose)
waypoints.append(grasp_pose)
(plan, fraction) = move_group.compute_cartesian_path(waypoints, 0.01, 0.0)
```

**MoveIt Task Constructor project:**
```bash
sudo apt install ros-humble-moveit-task-constructor-core
```

Build a pick-and-place task using MTC stages:
1. Move to pre-approach pose
2. Open gripper
3. Move to grasp pose
4. Close gripper
5. Retreat with object
6. Move to place pose
7. Open gripper

**GitHub project for Weeks 6–7:** A pick-and-place demo in Gazebo where a robot arm picks a colored box from a table and places it in a bin. This is one of the most requested portfolio projects in robotics job postings. Record a video and link it in the README.

**Resources for Week 7:**
- [MoveIt Task Constructor tutorials](https://moveit.picknik.ai/main/doc/examples/moveit_task_constructor/moveit_task_constructor_tutorial.html)
- [ros2_control documentation](https://control.ros.org/master/index.html)
- [PickNik Robotics blog](https://picknik.ai/blog) — the company behind MoveIt2, lots of practical articles

---

## Phase 5: Fleet Management (OpenRMF)
### Week 8 | ~15 hours | GitHub project: `openrmf-warehouse-demo`

Open Robotics Middleware Framework (OpenRMF) coordinates fleets of heterogeneous robots across shared spaces. It handles traffic management, task scheduling, and integration between different robot systems.

**Concepts to learn:**
- **rmf_traffic**: core traffic management library — handles schedules and conflict avoidance
- **rmf_fleet_adapter**: the bridge between OpenRMF and individual robot systems
- **Traffic editor**: GUI tool for drawing maps, defining lanes, lifts, doors
- **Task dispatcher**: receives task requests and assigns them to robots
- **API server**: REST/WebSocket API for sending tasks to the fleet
- **Robot modes**: full, read-only, and traffic-light integration levels

**Install OpenRMF:**
```bash
sudo apt install ros-humble-rmf-dev
# or build from source (recommended for learning):
git clone https://github.com/open-rmf/rmf.git
cd rmf && vcs import < rmf.repos
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
```

**Run the OpenRMF demos:**
```bash
# Hotel demo — multiple robots doing delivery tasks
git clone https://github.com/open-rmf/rmf_demos
cd rmf_demos
ros2 launch rmf_demos_gz office.launch.xml
```

Open the RMF Panel in the browser (localhost:3000) and dispatch tasks to the fleet.

**Practice tasks:**
1. Explore the office demo — dispatch delivery tasks, observe traffic negotiation
2. Use Traffic Editor to draw a custom map with lanes and waypoints
3. Integrate a single simulated robot as an RMF fleet adapter (full integration)
4. Dispatch tasks via the REST API:
```bash
curl -X POST http://localhost:7878/submit-task \
  -H "Content-Type: application/json" \
  -d '{"type": "delivery", "robot": "tinyRobot1", ...}'
```

**Key repos to study:**
- [open-rmf/rmf_demos](https://github.com/open-rmf/rmf_demos) — working examples
- [open-rmf/rmf_fleet_adapter](https://github.com/open-rmf/rmf_fleet_adapter) — how to integrate a robot
- [open-rmf/traffic_editor](https://github.com/open-rmf/traffic_editor) — map creation tool

**Resources for Week 8:**
- [OpenRMF Documentation](https://osrf.github.io/ros2multirobotbook/) — the multi-robot book, comprehensive
- [OpenRMF GitHub organization](https://github.com/open-rmf)
- [ROSCon talks on OpenRMF](https://vimeo.com/showcase/9954564) — search "RMF" for practical talks

---

## Phase 6: Capstone Project
### Week 9 | ~15 hours | GitHub project: `robotics-capstone`

Integrate everything into one cohesive system. This project is what you lead with in interviews.

**Project concept: Automated Warehouse Robot System**

A simulated warehouse where:
- A fleet of mobile robots (Nav2) navigate shared spaces managed by OpenRMF
- One or more robot arms (MoveIt2) pick items at stations
- All robots operate in a Gazebo simulated environment
- Tasks are dispatched via a simple web interface

**What to build:**

1. **Gazebo world**: A warehouse with shelves, pick stations, delivery zones, and multiple robots
2. **Navigation layer**: Each mobile robot runs Nav2 for autonomous navigation
3. **Fleet layer**: OpenRMF manages traffic so robots don't collide in shared corridors
4. **Manipulation layer**: A stationary arm at the pick station uses MoveIt2 to pick items when a mobile robot arrives
5. **Task orchestration**: A simple Python script that dispatches the full pick-and-deliver workflow

**Minimum viable version (if time is short):** Two mobile robots navigating a shared space under OpenRMF control, with Nav2. Add the arm as a stretch goal.

**What makes this portfolio-ready:**
- Clean README with architecture diagram
- GIF or video of the system running
- Launch file that starts everything with one command
- Documented configuration files with explanations

---

## Key GitHub Repositories to Watch and Study

Understanding real codebases is as important as writing your own. Star and read these:

| Repo | Why |
|------|-----|
| [ros2/examples](https://github.com/ros2/examples) | Official ROS2 code examples in Python and C++ |
| [nav2/navigation2](https://github.com/ros-planning/navigation2) | Nav2 source — read the behavior trees and plugin interfaces |
| [moveit/moveit2](https://github.com/moveit/moveit2) | MoveIt2 source — especially the tutorials folder |
| [open-rmf/rmf_demos](https://github.com/open-rmf/rmf_demos) | Working OpenRMF examples |
| [ros-controls/ros2_control](https://github.com/ros-controls/ros2_control) | Hardware interface layer — bridges software to real hardware |
| [Articulated Robotics](https://github.com/joshnewans/articubot_one) | Real-world URDF + Nav2 setup, great reference |
| [TurtleBot3](https://github.com/ROBOTIS-GIT/turtlebot3) | Gold standard ROS2 mobile robot implementation |

---

## Interview Prep: What Hireable Means in Robotics Software

By week 9, you should be able to answer these confidently:

**ROS2:**
- Explain the difference between topics, services, and actions — and when to use each
- How does TF2 work and why is it necessary?
- What is the DDS layer in ROS2 and why does it matter?
- How do you debug a misbehaving node? (`ros2 topic echo`, `rqt_graph`, logs)

**Gazebo:**
- What is the difference between URDF and SDF?
- How does the ros_gz_bridge work?
- How do you add a sensor to a simulated robot?

**Nav2:**
- Explain the role of global vs. local costmaps
- What is a behavior tree and how does Nav2 use it?
- How does SLAM work at a conceptual level?
- How would you tune a robot that keeps oscillating around its goal?

**MoveIt2:**
- What is inverse kinematics and when does it fail?
- How does the planning scene work?
- What is MoveIt Task Constructor?
- Explain the difference between joint-space and Cartesian planning

**OpenRMF:**
- What problem does OpenRMF solve that Nav2 alone doesn't?
- How does traffic negotiation work between robots?
- What is a fleet adapter?

---

## Weekly Time Budget Breakdown

| Week | Phase | Hours | Deliverable |
|------|-------|-------|-------------|
| 1 | ROS2 Fundamentals I | 15 | Publisher, subscriber, ROS2 graph understanding |
| 2 | ROS2 Fundamentals II | 15 | Services, actions, launch files, URDF intro |
| 3 | Gazebo Simulation | 15 | Custom robot in Gazebo with sensors |
| 4 | Nav2 Foundations | 15 | SLAM mapping demo with TurtleBot3 |
| 5 | Nav2 Advanced | 15 | Autonomous waypoint navigation on own robot |
| 6 | MoveIt2 Foundations | 15 | Panda arm motion planning, custom robot config |
| 7 | MoveIt2 Advanced | 15 | Pick and place demo with collision objects |
| 8 | OpenRMF | 15 | Multi-robot fleet demo |
| 9 | Capstone | 15 | Integrated warehouse system |

---

## Resource Quick Reference

| Resource | Type | Cost | Best for |
|----------|------|------|---------|
| [docs.ros.org/en/humble](https://docs.ros.org/en/humble) | Docs | Free | Official reference |
| [navigation.ros.org](https://navigation.ros.org) | Docs | Free | Nav2 everything |
| [moveit.picknik.ai](https://moveit.picknik.ai/main/index.html) | Docs | Free | MoveIt2 everything |
| [osrf.github.io/ros2multirobotbook](https://osrf.github.io/ros2multirobotbook/) | Book | Free | OpenRMF deep dive |
| [Robotics Backend (YouTube)](https://www.youtube.com/@RoboticsBackEnd) | Video | Free | Best ROS2/Nav2/MoveIt2 tutorials |
| [Articulated Robotics (YouTube)](https://www.youtube.com/@ArticulatedRobotics) | Video | Free | URDF + Gazebo + Nav2 |
| [The Construct](https://www.theconstructsim.com) | Platform | Free tier | Browser-based ROS2 environment |
| [ROS2 for Beginners (Udemy)](https://www.udemy.com/course/ros2-for-beginners/) | Course | ~$15 | Structured video curriculum |
| [ROSCon talks (Vimeo)](https://vimeo.com/showcase/9954564) | Talks | Free | Real-world use cases, advanced topics |
| [ROS Discourse](https://discourse.ros.org) | Community | Free | Ask questions, follow announcements |
| [ROS Answers](https://robotics.stackexchange.com) | Community | Free | Debugging help |

---

## Tips for Staying on Track

**Keep a learning log.** Each day, write 2–3 sentences about what you built and what confused you. This doubles as interview prep material.

**Commit to GitHub every session.** Even half-finished code. The commit history tells your story.

**Don't skip phases.** MoveIt2 will not make sense without ROS2 fundamentals. Nav2 will break in mysterious ways if you don't understand TF2.

**Simulation bugs are normal.** Gazebo is finicky. When something doesn't work, check: Is the bridge running? Is the TF tree connected? Is the topic name correct? Use `rqt_graph` and `ros2 topic list` obsessively.

**Join the community.** The [ROS Discord](https://discord.com/invite/ros) and ROS Discourse forums are active and helpful. Don't spend more than 30 minutes stuck before asking.

**Record your demos.** Use `kazam` or `simplescreenrecorder` on Linux to capture GIFs of your robots. These are the most compelling thing you can add to a portfolio.
