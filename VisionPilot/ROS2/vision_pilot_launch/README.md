# VisionPilot Launch Package

This package provides comprehensive launch files for running VisionPilot with web interfaces and systemd service management.

## Launch Files

### `vision_pilot_web.launch.xml`
Complete VisionPilot pipeline with full web interface capabilities:
- VisionPilot AI models (SceneSeg, DomainSeg, Scene3D)
- Foxglove Bridge for web dashboard
- Web Video Server for HTTP streaming
- ROSBridge Server (optional)
- Image compression for better web performance

### `vision_pilot_simple.launch.xml`
Simplified VisionPilot setup with basic web interface:
- VisionPilot pipeline
- Foxglove Bridge
- Web Video Server

## Usage

### Traditional ROS2 Launch

```bash
# Source the workspace
cd autoware-pov/VisionPilot/ROS2
source install/setup.sh

# Run complete web interface
ros2 launch vision_pilot_launch vision_pilot_web.launch.xml \
    video_path:=/path/to/video.mp4 \
    pipeline:=scene_seg

# Run simple interface
ros2 launch vision_pilot_launch vision_pilot_simple.launch.xml \
    video_path:=/path/to/video.mp4 \
    pipeline:=domain_seg
```

### Systemd Service Management (Recommended)

Using the project Makefile with ros2systemd:

```bash
# Start web service
make run-web-service VIDEO=/path/to/video.mp4 PIPELINE=scene_seg

# Start simple service
make run-simple-service VIDEO=/path/to/video.mp4 PIPELINE=scene_3d

# Check service status
make status-vision-services

# View service logs
make logs-vision-services SERVICE=vision-pilot-web

# Stop all services
make stop-vision-services

# Remove all services
make remove-vision-services
```

## Launch Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `video_path` | - | **Required**: Path to input video file |
| `pipeline` | `scene_seg` | Pipeline type: `scene_seg`, `domain_seg`, or `scene_3d` |
| `enable_foxglove` | `true` | Enable Foxglove Bridge |
| `enable_web_video` | `true` | Enable Web Video Server |
| `enable_rosbridge` | `false` | Enable ROSBridge Server |
| `foxglove_port` | `8765` | Foxglove WebSocket port |
| `web_video_port` | `8080` | Web Video Server HTTP port |
| `rosbridge_port` | `9090` | ROSBridge WebSocket port |

## Web Access

After launching, access VisionPilot via:

- **Foxglove Dashboard**: https://app.foxglove.dev/
  - Connect to: `ws://ROBOT_IP:8765`
- **Video Streams**: `http://ROBOT_IP:8080/`
  - Raw feed: `/stream_viewer?topic=/sensors/video/image_raw`
  - Visualization: `/stream_viewer?topic=/autoseg/scene_seg/viz`
- **ROSBridge** (if enabled): `ws://ROBOT_IP:9090`

## ROS2 Topics

| Topic | Type | Description |
|-------|------|-------------|
| `/sensors/video/image_raw` | sensor_msgs/Image | Input video feed |
| `/autoseg/scene_seg/mask` | sensor_msgs/Image | Scene segmentation masks |
| `/autoseg/domain_seg/mask` | sensor_msgs/Image | Domain segmentation masks |
| `/auto3d/scene_3d/depth_map` | sensor_msgs/Image | Depth estimation |
| `/autoseg/scene_seg/viz` | sensor_msgs/Image | Scene segmentation visualization |
| `/autoseg/domain_seg/viz` | sensor_msgs/Image | Domain segmentation visualization |
| `/auto3d/scene_3d/viz` | sensor_msgs/Image | Depth visualization |
| `/*_compressed` | sensor_msgs/CompressedImage | Compressed versions for web |

## Dependencies

Required packages (automatically handled by package.xml):
- `foxglove_bridge` - WebSocket bridge for Foxglove
- `web_video_server` - HTTP video streaming
- `rosbridge_server` - WebSocket bridge for custom interfaces
- `image_transport` - Image format conversion
- `sensors`, `models`, `visualization` - VisionPilot core packages

## Systemd Integration

This package integrates with [ros2systemd](https://github.com/jerry73204/ros2systemd) for robust service management:

- **Automatic restart** on failure
- **System integration** with systemd
- **Logging** via journald
- **Resource management** and monitoring
- **Boot-time startup** (optional)

Services are created as user services and can be managed using standard systemd commands or the provided Makefile targets.

## Examples

### Scene Segmentation with Full Web Interface
```bash
make run-web-service VIDEO=/home/jetson/videos/highway.mp4 PIPELINE=scene_seg
```

### Depth Estimation with Simple Interface
```bash
make run-simple-service VIDEO=/home/jetson/videos/driving.mp4 PIPELINE=scene_3d
```

### Monitor Running Services
```bash
make status-vision-services
make logs-vision-services SERVICE=vision-pilot-web
```

This provides a complete web-accessible VisionPilot deployment suitable for remote monitoring, development, and integration with the Autoware POV autonomous driving stack.