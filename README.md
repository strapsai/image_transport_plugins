Content created by Yaoyu's AI assistant. Use with care.

# Image Transport Plugins

`image_transport_plugins` is a meta-package containing multiple plugins for the standard ROS 2 `image_transport` package. These plugins provide common image compression and decompression capabilities (e.g., Compressed, Theora, Zstd) for efficient image transmission over different network conditions.

## Overview

The package bundles several transport plugins that extend `image_transport` to support specific encoding and decoding formats:
- **Compressed**: Basic JPEG/PNG compression.
- **Theora**: Lossy video compression using the Theora codec.
- **Zstd**: High-performance lossless compression using Zstandard.
- **Compressed Depth**: Specialized compression for depth images.

## Key Plugins

### Compressed Image Transport
A plugin for basic JPEG and PNG compression, widely used for standard camera feeds.
- Source: `compressed_image_transport/`

### Theora Image Transport
A plugin for low-bandwidth video streams using the Theora codec.
- Source: `theora_image_transport/`

### Zstd Image Transport
A plugin for efficient lossless compression using the Zstd algorithm.
- Source: `zstd_image_transport/`

### Compressed Depth Image Transport
A specialized plugin for compressing 16-bit or 32-bit floating-point depth images.
- Source: `compressed_depth_image_transport/`

## Repository Structure

- `compressed_depth_image_transport/`: Specialized depth image compression.
- `compressed_image_transport/`: Standard JPEG/PNG compression.
- `theora_image_transport/`: Theora video compression.
- `zstd_image_transport/`: Zstd lossless compression.

## Prerequisites

- **ROS 2 Humble / Rolling**
- **C++17**
- **Dependencies**:
  - `rclcpp`
  - `sensor_msgs`
  - `cv_bridge`
  - `image_transport`
  - `theora`, `ogg` (for Theora plugin)
  - `zstd` (for Zstd plugin)

## Installation

1. Clone the repository into your ROS 2 workspace:
   ```bash
   cd ~/ros2_ws/src
   git clone https://github.com/strapsai/image_transport_plugins.git
   ```

2. Build the package:
   ```bash
   cd ~/ros2_ws
   colcon build --packages-up-to image_transport_plugins
   ```

## Usage

Once installed, the plugins are automatically discovered by `image_transport`. You can list available transports using:
```bash
ros2 run image_transport list_transports
```

To use a specific transport, specify it as a parameter:
```bash
ros2 run image_transport republish raw in:=/camera/image_raw compressed out:=/camera/image_compressed
```

## License

This project is licensed under the BSD-3-Clause License - see the [LICENSE](LICENSE) file for details.
