# 🤖 ROS2 Vision-Guided Robot

A ROS2 (Humble/Iron) package that combines an **OpenCV lane-detection node** (Python) with a **C++ velocity controller** for a differential-drive robot. Includes obstacle avoidance via depth camera or LiDAR.

---

## Architecture

```
┌─────────────────────┐     /camera/image_raw      ┌──────────────────┐
│   Camera Driver     │ ──────────────────────────► │   vision_node.py │
└─────────────────────┘                             └────────┬─────────┘
                                                             │ /vision/steering_cmd
┌─────────────────────┐     /camera/depth/image_raw ┌────────▼─────────┐
│  Depth Cam / LiDAR  │ ──────────────────────────► │ obstacle_node.py │
└─────────────────────┘                             └────────┬─────────┘
                                                             │ /vision/obstacle_dist
                                                    ┌────────▼──────────────┐
                                                    │ velocity_controller   │  (C++)
                                                    └────────┬──────────────┘
                                                             │ /cmd_vel
                                                    ┌────────▼──────────────┐
                                                    │   Robot / Simulator   │
                                                    └───────────────────────┘
```

---

## Prerequisites

| Tool | Version |
|------|---------|
| ROS2 | Humble or Iron |
| OpenCV | ≥ 4.8 |
| cv_bridge | ROS2 package |
| Python | ≥ 3.10 |

---

## Build & Install

```bash
# In your ROS2 workspace
cd ~/ros2_ws/src
git clone https://github.com/<you>/ros2-vision-guided-robot.git

cd ~/ros2_ws
colcon build --packages-select vision_guided_robot
source install/setup.bash
```

---

## Usage

```bash
# Launch everything (with depth camera)
ros2 launch vision_guided_robot robot_launch.py

# Use LaserScan instead of depth camera
ros2 launch vision_guided_robot robot_launch.py use_depth:=false

# Disable debug image stream
ros2 launch vision_guided_robot robot_launch.py debug:=false
```

### Individual nodes

```bash
# Vision node only
ros2 run vision_guided_robot vision_node.py

# Velocity controller only
ros2 run vision_guided_robot velocity_controller
```

---

## Topics

| Topic | Type | Direction | Description |
|-------|------|-----------|-------------|
| `/camera/image_raw` | `sensor_msgs/Image` | Input | RGB camera |
| `/camera/depth/image_raw` | `sensor_msgs/Image` | Input | Depth (32FC1) |
| `/scan` | `sensor_msgs/LaserScan` | Input | 2D LiDAR |
| `/odom` | `nav_msgs/Odometry` | Input | Robot odometry |
| `/vision/steering_cmd` | `std_msgs/Float32` | Internal | [-1, 1] steering |
| `/vision/obstacle_dist` | `std_msgs/Float32` | Internal | metres |
| `/vision/debug_image` | `sensor_msgs/Image` | Output | Annotated frame |
| `/cmd_vel` | `geometry_msgs/Twist` | Output | Robot velocity |

---

## Parameters

### vision_node
| Param | Default | Description |
|-------|---------|-------------|
| `roi_top_frac` | `0.55` | Upper boundary of ROI |
| `canny_lo/hi` | `50/150` | Canny edge thresholds |
| `hough_thresh` | `40` | Hough line threshold |
| `debug` | `true` | Publish debug image |

### velocity_controller
| Param | Default | Description |
|-------|---------|-------------|
| `base_speed` | `0.25` m/s | Cruise speed |
| `max_angular` | `1.2` rad/s | Max turn rate |
| `stop_dist` | `0.40` m | Emergency stop distance |
| `slow_dist` | `1.00` m | Slow-down threshold |

---

## Simulation (Gazebo)

```bash
# With TurtleBot3 (requires turtlebot3 packages)
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py

# In a second terminal
ros2 launch vision_guided_robot robot_launch.py
```

---

## Extending

- **Replace lane detector** — swap `_compute_steering()` in `vision_node.py` with a YOLO-based road-surface segmenter
- **Add PID** — integrate a PID controller in `velocity_controller.cpp` using `/odom` feedback for closed-loop speed control
- **3-D obstacles** — point-cloud processing via `pcl_ros` instead of flat depth ROI
