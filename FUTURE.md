# FUTURE.md — image_transport_plugins

> ROS2 image_transport plugin collection providing JPEG/PNG compressed, compressed depth, and Theora video stream transports as transparent drop-in alternatives to raw image topics.
> Last updated: 2026-03-05

## Purpose
Extends `image_transport` with bandwidth-efficient transport backends. Any node using `image_transport` pub/sub automatically gains access to these plugins without code changes — the transport is selected at runtime via topic remapping or parameter. Critical for network-constrained robot deployments (WiFi, LTE).

## Sub-Packages

| Package | Description |
|---------|-------------|
| `compressed_image_transport` | JPEG/PNG/TIFF compression for color and mono images |
| `compressed_depth_image_transport` | Inverse-depth quantization compression for depth images |
| `theora_image_transport` | Theora video codec streaming (low bitrate, lossy) |

## Nodes
These packages contain **no standalone nodes**. They register `image_transport` plugins via `pluginlib` (XML manifests). The pub/sub logic is injected transparently into any node using `image_transport::ImageTransport`.

Plugin registration files:
- `compressed_image_transport/compressed_plugins.xml`
- `compressed_depth_image_transport/compressed_depth_plugins.xml`
- `theora_image_transport/theora_plugins.xml`

## Design Pattern

**pluginlib transport pattern**: Each package exports a `PublisherPlugin` and `SubscriberPlugin` class registered via `pluginlib`. `image_transport` discovers these at runtime.

To use compressed transport in any node:
```cpp
// No code change needed — topic suffix selects transport automatically:
// /camera/image_raw               → raw transport
// /camera/image_raw/compressed    → CompressedImage transport
// /camera/image_raw/compressedDepth → CompressedDepth transport
// /camera/image_raw/theora        → Theora transport
```

From CLI:
```bash
# Force compressed transport for a subscriber:
ros2 run image_view image_view --ros-args -r image:=/camera/image_raw \
  -p image_transport:=compressed
```

## ROS Interfaces

### compressed_image_transport
| Topic Suffix | Msg Type | Notes |
|---|---|---|
| `/compressed` | `sensor_msgs/CompressedImage` | Publisher output / Subscriber input |

### compressed_depth_image_transport
| Topic Suffix | Msg Type | Notes |
|---|---|---|
| `/compressedDepth` | `sensor_msgs/CompressedImage` | Uses custom `ConfigHeader` prepended to data for format/quantization info |

### theora_image_transport
| Topic Suffix | Msg Type | Notes |
|---|---|---|
| `/theora` | `theora_image_transport/Packet` | OGG Theora packets |

### Parameters

#### compressed_image_transport Publisher
| Param | Default | Description |
|-------|---------|-------------|
| `format` | `jpeg` | Compression format: `jpeg`, `png`, `tiff` |
| `jpeg_quality` | `95` | JPEG quality (0–100) |
| `png_level` | `3` | PNG compression level (0–9) |

#### compressed_depth_image_transport Publisher
Uses inverse-depth quantization (not JPEG). The `ConfigHeader` struct is prepended to the compressed byte array:
- `format`: always `INV_DEPTH`
- `depthParam[2]`: quantization coefficients stored in the stream header

#### theora_image_transport Publisher
| Param | Default | Description |
|-------|---------|-------------|
| `quality` | 31 | Theora quality (0–63) |
| `keyframe_frequency` | 64 | Keyframe interval |
| `optimize_for` | 0 | 0=quality, 1=bitrate |
| `target_bitrate` | 800000 | Target bitrate (optimize_for=1) |

## Launch Files
No launch files. Plugins are loaded automatically by `image_transport` when installed.

## Config Files
- `*_plugins.xml`: `pluginlib` registration manifests. Lists `PublisherPlugin` / `SubscriberPlugin` class names. Must match `package.xml` `<export>` entries.

## Key Dependencies

| Dependency | Usage |
|------------|-------|
| `image_transport` | Plugin interface base classes; auto-discovers these plugins |
| `cv_bridge` | Image ↔ OpenCV conversion for JPEG/PNG encoding |
| `libopencv-dev` | `cv::imencode`/`cv::imdecode` for compression |
| `libtheora-dev` | Theora codec (encode/decode) |
| `libogg-dev` | OGG container for Theora stream |

## Build Notes
Standard `ament_cmake`. No special flags. Must be built alongside `image_transport`:
```bash
colcon build --packages-up-to compressed_image_transport compressed_depth_image_transport theora_image_transport
```

## Known Issues

| Severity | Description |
|----------|-------------|
| Info | Upstream community package (ros-perception). Do not fork — use as-is. |
| Low | Theora introduces latency from keyframe interval; not suitable for low-latency control feedback. |
| Low | `compressed_depth_image_transport` uses custom quantization — not compatible with standard JPEG depth compression used by some camera drivers (e.g., RealSense). Verify round-trip fidelity before relying on depth values. |
