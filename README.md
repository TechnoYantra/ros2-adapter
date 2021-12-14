## ROS2 adapter functionality

For a full list of Formant telemetry types, see: https://formant.readme.io/docs/how-telemetry-streams-work

For a list of what is currently supported by the Formant ROS2 Adapter:

### Basic datapoints
- Numeric (UInt, Int, and Float types) :heavy_check_mark:
- Text (String, Char) :heavy_check_mark:
- Bitset (Bool) :heavy_check_mark:
- Location (NavSatFix) :heavy_check_mark:
- Battery (Battery) :heavy_check_mark:

All other input types will be ingested as JSON.

### Input from multiple fields
- Bitset (multiple Bool inputs) :heavy_check_mark:
- Numericset (multiple UInt, Int, or Float inputs) :heavy_check_mark:

### Rich datapoints
- Point Clouds (PointCloud2, LaserScan) :heavy_check_mark:
- Images and Video (CompressedImage) :heavy_check_mark:
- Localization (Map, Odometry, Path, etc.) TODO 
- Transform Tree (/tf, /tf_static) TODO

## Configuring the ROS2 Adapter

### Basic configuration

Add a file `config.json` that contains each ROS topic name to ingest as telemetry under the "streams" key. e.g.

```
{
    "streams": [
        {
            "topic": "/cmd_vel"
        },
        {
            "topic": "/rgb/image_raw/compressed"
        },
        {
            "topic": "/depth/points"
        },
        {
            "topic": "/base_scan",
            "stream": "Single-channel lidar scan"
        }
    ]
}
```

Topics will automatically be ingested as their corresponding Formant type:

| ROS topic type                               | Formant datapoint type |
|----------------------------------------------|------------------------|
| Bool, message with bool-valued message paths | bitset                 |
| Message with numeric-valued message paths    | numeric set            |
| Char, String                                 | text                   |
| Float, Int, Uint                             | numeric                |
| NavSatFix                                    | location               |
| LaserScan, PointCloud2                       | point cloud            |
| CompressedImage                              | image, video           |

By default, stream name is automatically configured from the topic. (e.g. "/base/cmd_vel" -> "base.cmd_vel") The `"stream"` configuration can be set to change the stream name of ingested datapoints manually.

### Message path configuration

`messagePath` and `messagePaths` can be set to ingest specific values, or multiple values.

```
{
    "streams": [
        {
            "topic": "/battery",
            "stream": "Battery Voltage"
            "messagePath": "voltage"
        },
        {
            "formantType": "numericset",
            "topic": "/battery",
            "stream": "Battery Set"
            "messagePaths": ["voltage", "current", "charge"],
            "units": ["volts", "A", "Ah"]
        },
        {
            "formantType": "bitset",
            "topic": "/RegionOfInterest",
            "messagePaths": ["do_rectify"]
        },
    ]
}
```

Setting the `"formantType"` to `"numericset"` or `"bitset"` and specifying multiple values in `"messagePaths"` will ingest multiple fields from a given ROS topic into a single, multi-valued datapoint in Formant.

