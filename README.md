# 3D Room Scanner with Intel RealSense D435

A ROS 2 Humble-based 3D room scanning system for Raspberry Pi with Intel RealSense D435 depth camera, touchscreen visualization, and optional IMU integration.

## System Overview

### Hardware Requirements
- Raspberry Pi 4 (4GB+ RAM recommended)
- Intel RealSense D435 depth camera
- Touchscreen display
- External IMU (optional, for future integration)
- Ubuntu 22.04 (64-bit recommended for ROS 2 Humble)

### Software Stack
- **ROS 2 Humble Hawksbill**
- **Intel RealSense ROS 2 Wrapper** - Camera interface
- **RTABMap ROS** - Real-time point cloud mapping and SLAM
- **RViz2** - 3D visualization
- **PCL (Point Cloud Library)** - Point cloud processing

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    RealSense D435 Camera                     │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              realsense2_camera (ROS 2 Node)                  │
│  Publishes: /camera/color/image_raw                          │
│            /camera/depth/image_rect_raw                      │
│            /camera/depth/color/points                        │
│            /camera/color/camera_info                         │
└──────────┬──────────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────────┐
│              RTABMap or Octomap Server                       │
│  - Aggregates point clouds over time                         │
│  - Performs SLAM (optional)                                  │
│  - Publishes: /map (point cloud map)                         │
│               /octomap_full (3D occupancy grid)              │
└──────────┬──────────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────────┐
│                      RViz2 Visualization                     │
│  - Display live point cloud                                  │
│  - Display aggregated map                                    │
│  - Touch-friendly interface                                  │
└─────────────────────────────────────────────────────────────┘
```

## Installation

### 1. Install ROS 2 Humble

```bash
cd ~/mujoco_projects
./scripts/install_ros2_humble.sh
```

### 2. Install Dependencies

```bash
./scripts/install_dependencies.sh
```

### 3. Build Workspace

```bash
cd ros2_ws
colcon build --symlink-install
source install/setup.bash
```

### 4. Set Up USB Rules for RealSense

```bash
./scripts/setup_realsense_udev.sh
```

## Usage

### Quick Start (Manual)

```bash
# Terminal 1: Start the camera and mapping
ros2 launch room_scanner scanner.launch.py

# Terminal 2: Start RViz2 (if not auto-launched)
ros2 run rviz2 rviz2 -d ~/mujoco_projects/rviz/scanner_config.rviz
```

### Auto-Start on Boot

```bash
# Install systemd service
sudo ./scripts/install_service.sh

# Enable auto-start
sudo systemctl enable room-scanner

# Start now
sudo systemctl start room-scanner
```

### Saving Point Cloud Maps

While running, you can save the current map:

```bash
# Save point cloud to PCD file
ros2 service call /rtabmap/save_map std_srvs/srv/Empty

# Or use octomap save (if using octomap)
ros2 run octomap_server octomap_saver -f ~/maps/scan_$(date +%Y%m%d_%H%M%S).bt
```

### Configuration

Edit configuration files in `config/`:
- `camera_params.yaml` - RealSense camera settings
- `rtabmap_params.yaml` - Mapping parameters
- `octomap_params.yaml` - Octomap settings (alternative)

## Package Structure

```
mujoco_projects/
├── ros2_ws/                    # ROS 2 workspace
│   └── src/
│       └── room_scanner/       # Custom package
│           ├── launch/         # Launch files
│           ├── config/         # Configuration files
│           └── package.xml
├── scripts/                    # Installation scripts
├── rviz/                       # RViz configurations
└── config/                     # Additional configs
```

## ROS 2 Topics

### Published by RealSense
- `/camera/color/image_raw` - RGB image
- `/camera/depth/image_rect_raw` - Depth image
- `/camera/depth/color/points` - Point cloud (colored)
- `/camera/color/camera_info` - Camera calibration
- `/camera/depth/camera_info` - Depth camera calibration

### Published by Mapping Node
- `/rtabmap/mapData` - RTAB-Map data
- `/rtabmap/cloud_map` - Aggregated point cloud
- `/octomap_full` - Full 3D occupancy map (if using octomap)

## Performance Tuning for Raspberry Pi

The configuration files are optimized for Raspberry Pi 4:
- Reduced image resolution (640x480 default)
- Lower frame rate (15 FPS)
- Point cloud downsampling
- Memory-efficient mapping parameters

For better performance:
1. Overclock your Raspberry Pi (if using active cooling)
2. Use 64-bit Ubuntu for better performance
3. Reduce resolution further if needed in `config/camera_params.yaml`

## Future Enhancements

### IMU Integration
When adding an external IMU:
1. Install IMU ROS 2 driver (e.g., `imu_filter_madgwick`)
2. Update launch file to include IMU node
3. Configure RTABMap to fuse IMU data for better odometry

### Advanced Features
- **Loop Closure** - Already enabled in RTABMap for better mapping
- **Mesh Generation** - Add mesh_tools or Open3D integration
- **Export to other formats** - PLY, OBJ, STL for 3D printing

## Troubleshooting

### Camera not detected
```bash
# Check USB connection
lsusb | grep Intel

# Check RealSense firmware
rs-fw-update -l
```

### Performance issues
- Reduce resolution in `config/camera_params.yaml`
- Lower frame rate
- Disable RGB processing if only depth needed
- Increase point cloud voxel size

### RViz2 crashes or slow
- Use the lightweight config: `rviz/scanner_minimal.rviz`
- Disable grid and other heavy visualizations
- Reduce point cloud display size

## Resources

- [RealSense ROS 2 Wrapper](https://github.com/IntelRealSense/realsense-ros)
- [RTABMap ROS](https://github.com/introlab/rtabmap_ros)
- [ROS 2 Humble Documentation](https://docs.ros.org/en/humble/)

## License

See LICENSE file.
