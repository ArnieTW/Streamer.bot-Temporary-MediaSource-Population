# Streamer.bot Temporary MediaSource Population

A comprehensive Streamer.bot C# action that creates temporary sources in OBS Studio with intelligent type detection, automatic filtering, and cleanup. Supports multiple source types including media files, images, text, and text files. Perfect for overlay effects, alerts, notifications, or any temporary content that needs to appear for a specific duration.

## Features

- ✨ **Multi-Source Support**: Media files (video/audio), images, text content, and text files
- 🧠 **Intelligent Detection**: Automatically determines source type from file extensions
- 🎨 **Chroma Key Support**: Built-in chroma key filtering with customizable settings
- 📝 **Advanced Text Sources**: Modern text_gdiplus_v3 with full font customization
- ⏰ **Timed Cleanup**: Automatically removes sources after a specified duration
- 📐 **Flexible Positioning**: Customizable size and position for each source
- 🔊 **Smart Audio Monitoring**: Configurable audio output for media sources only
- 🔧 **Parameter Driven**: Highly configurable through Streamer.bot arguments
- 🎯 **Conflict Prevention**: Uses GUID-based naming to avoid source name conflicts
- 📝 **Comprehensive Logging**: Built-in logging for debugging and monitoring
- 🏗️ **Clean Architecture**: Well-structured code using FontSettings class and enum-based dispatching

## Requirements

### Software Dependencies
- **Streamer.bot** (Latest version recommended)
- **OBS Studio** with WebSocket plugin enabled
- **.NET Framework** (Streamer.bot compatible version)

### NuGet Packages
- `Newtonsoft.Json` - For JSON parsing and manipulation

## Installation

### Quick Setup (Simple)

1. Copy the C# code from `Execute C#` file
2. Create a new C# action in Streamer.bot
3. Paste the code into the action editor
4. Ensure Newtonsoft.Json is available in your Streamer.bot environment
5. Configure the action arguments (see Parameters section below)

### Full Setup (Recommended)

This method provides better organization and reusability by separating the code logic from parameter configuration.

#### Step 1: Create the Core C# Function

1. **Create New Action**
   - Right-click in Actions tab → "Add Action"
   - Name: `TempMediaSource_Core`
   - Trigger: **None** (No Trigger)

2. **Add C# Code Sub-Action**
   - Add Sub-Action → "Core" → "C#" → "Execute C# Code"
   - Paste the complete C# code from `Execute C#` file
   - **Important**: In Code Editor Settings, set **Name** field to: `TempMediaSourceFunction`
   - Click "Compile" to ensure no errors

#### Step 2: Create Parameter Configuration Actions

Create individual actions for each media type you want to use. Repeat this step for each different media source:

1. **Create Media-Specific Action**
   - Right-click in Actions tab → "Add Action"
   - Name: `TempMedia_FollowAlert` (or your preferred name)
   - Trigger: **None** (No Trigger)

2. **Configure Parameters**
   - Add Sub-Action → "Core" → "Variables" → "Set Argument"
   - Set each parameter (repeat for all needed parameters):
     ```
     BlinkContent: "D:\Alerts\follow-alert.mp4"
     BlinkTaskLength: 3
     BlinkWidth: 400
     BlinkHeight: 200
     BlinkPosX: 960
     BlinkPosY: 100
     BlinkChroma: 0x00FF00
     BlinkChromaSim: 400
     BlinkChromaSmooth: 80
     BlinkAudioMonitor: true
     ```

3. **Add Execute Code Sub-Action**
   - Add Sub-Action → "Core" → "C#" → "Execute C# Method"
   - In the dropdown, select: `TempMediaSourceFunction` (the named function from Step 1)
   - This should be the **last** sub-action in the list

4. **Repeat for Multiple Media Types**
   - `TempMedia_SubAlert`
   - `TempMedia_DonationAlert`
   - `TempMedia_HostAlert`
   - etc.

#### Step 3: Create Trigger Actions

Create the actions that will actually be triggered by events:

1. **Create Triggered Action**
   - Right-click in Actions tab → "Add Action"
   - Name: `OnFollow` (or your event name)
   - Trigger: **Set your desired trigger** (e.g., Twitch Follow, Channel Point Reward, etc.)

2. **Add Run Action Sub-Action**
   - Add Sub-Action → "Core" → "Actions" → "Run Action"
   - Select Action: `TempMedia_FollowAlert` (the parameter action from Step 2)

3. **Repeat for All Events**
   - Create `OnSubscription` → Run `TempMedia_SubAlert`
   - Create `OnDonation` → Run `TempMedia_DonationAlert`
   - Create `OnHost` → Run `TempMedia_HostAlert`
   - etc.

#### Benefits of Full Setup

- **Reusability**: One core function, multiple configurations
- **Organization**: Clear separation of logic and parameters
- **Maintainability**: Update code in one place, affects all media sources
- **Flexibility**: Easy to add new media types without duplicating code
- **Testing**: Can test individual parameter sets without triggers

## Sizing Behavior

The system handles source sizing intelligently:

- **Default (0,0)**: Sources appear at their native dimensions - OBS determines optimal size
- **Custom Sizing**: Both width AND height must be specified for custom dimensions
- **Position Only**: Set BlinkPosX/BlinkPosY without width/height for native-sized positioning

### Sizing Examples

#### Native Size (Recommended)
```
BlinkContent: "video.mp4"
// BlinkWidth and BlinkHeight omitted or set to 0
// Source appears at native resolution
```

#### Custom Dimensions
```
BlinkContent: "video.mp4"
BlinkWidth: 1920     // Both required for custom sizing
BlinkHeight: 1080    // Both required for custom sizing
```

#### Positioned Native Size
```
BlinkContent: "overlay.png"
BlinkPosX: 1600      // Position without resizing
BlinkPosY: 50
// Width/Height = 0, so native size is preserved
```

## Supported Source Types

The system automatically detects source types based on file extensions:

| Source Type | Extensions | Description |
|-------------|------------|-------------|
| **Media** | `.mp4`, `.avi`, `.mov`, `.mkv`, `.wmv`, `.flv`, `.webm`, `.m4v`, `.3gp`, `.mpg`, `.mpeg`, `.mp3`, `.wav`, `.ogg`, `.m4a`, `.aac`, `.flac`, `.opus` | Video & audio files using ffmpeg_source |
| **Image** | `.jpg`, `.jpeg`, `.png`, `.gif`, `.bmp`, `.tiff`, `.tga`, `.webp` | Static images using image_source |
| **Text File** | `.txt`, `.rtf`, `.log`, `.md`, `.json`, `.xml`, `.csv` | Text content loaded from file using text_gdiplus_v3 |
| **Text** | N/A | Direct text content using text_gdiplus_v3 |

### Force Type Override (Optional)

**Automatic detection works for 99% of use cases.** You only need `BlinkForceType` for special situations:

- `Media` - Force media source (ffmpeg_source) for unusual file extensions
- `Image` - Force image source (image_source) for custom image formats  
- `TextFile` - Force text from file (text_gdiplus_v3) when path should be read as file
- `Text` - Force direct text content (text_gdiplus_v3) when content looks like a file path

**When to use:** Unusual file extensions, treating file paths as literal text, or custom formats.

## Parameters

Configure these arguments in your Streamer.bot action:

### Core Parameters
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `BlinkContent` | String | `""` | **Required** - File path or text content (primary parameter) |
| `BlinkVidPath` | String | `""` | **Legacy** - Backward compatibility only, use BlinkContent instead |
| `BlinkTaskLength` | Integer | `10` | Duration in seconds before auto-removal |
| `BlinkForceType` | String | `""` | **Optional** - Override automatic type detection (rarely needed) |
| `BlinkWidth` | Integer | `0` | **Required for custom sizing** - Source width in pixels (0 = native size) |
| `BlinkHeight` | Integer | `0` | **Required for custom sizing** - Source height in pixels (0 = native size) |
| `BlinkPosX` | Integer | `100` | X position on screen |
| `BlinkPosY` | Integer | `100` | Y position on screen |

### Media/Image Parameters
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `BlinkChroma` | Integer/Hex | `-1` | Chroma key color (use -1 to disable chroma key entirely) |
| `BlinkChromaSim` | Integer | `400` | Chroma key similarity threshold |
| `BlinkChromaSmooth` | Integer | `80` | Chroma key smoothness value |
| `BlinkAudioMonitor` | Boolean | `false` | Enable audio monitoring (media only) |
| `BlinkVolume` | Integer (0-100) | `-1` | Set media source volume percent (media only; -1 = unchanged) |
| `BlinkTracks` | String | `""` | Enable recording tracks: CSV like `"1,2,5"`, or `"all"`/`"none"` (media only) |

### Text Parameters
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `BlinkFontFamily` | String | `"Arial"` | Font family name |
| `BlinkFontSize` | Integer | `48` | Font size in points |
| `BlinkTextColor` | Integer/Hex | `0xFFFFFF` | Text color (white) |
| `BlinkFontStyle` | String | `""` | Font style (Bold, Italic, etc.) |
| `BlinkBgColor` | Integer/Hex | `0x000000` | Background color (black) |
| `BlinkBgOpacity` | Integer | `0` | Background opacity (0-100) |
| `BlinkEnableBg` | Boolean | `false` | Enable background rendering |

### Parameter Examples

#### Media File (Auto-detected)
```
BlinkContent: "D:\Videos\alert.mp4"
BlinkTaskLength: 5
BlinkChroma: 0x00FF00  (or 65280 for green, or -1 to disable)
BlinkWidth: 1920       (required for custom sizing)
BlinkHeight: 1080      (required for custom sizing)
BlinkPosX: 0
BlinkPosY: 0
BlinkChromaSim: 300
BlinkChromaSmooth: 100
BlinkAudioMonitor: true
BlinkVolume: 70        (70% volume)
BlinkTracks: "1,2"     (enable tracks 1 and 2)
```

#### Media File (Native Size)
```
BlinkContent: "D:\Videos\alert.mp4"
BlinkTaskLength: 5
BlinkChroma: 0x00FF00
BlinkPosX: 960         (positioned but native-sized)
BlinkPosY: 100
BlinkAudioMonitor: true
```

#### Image File (Auto-detected)
```
BlinkContent: "D:\Images\notification.png"
BlinkTaskLength: 3
BlinkChroma: 0xFF00FF  (magenta chroma key)
BlinkWidth: 400        (custom sizing)
BlinkHeight: 200
BlinkPosX: 960
BlinkPosY: 100
```

#### Image Without Chroma Key (Native Size)
```
BlinkContent: "D:\Images\overlay.png"
BlinkTaskLength: 5
BlinkChroma: -1        (disables chroma key completely)
BlinkPosX: 560         (positioned at native size)
BlinkPosY: 240
```

#### Text File (Auto-detected)
```
BlinkContent: "D:\Messages\welcome.txt"
BlinkTaskLength: 8
BlinkFontFamily: "Segoe UI"
BlinkFontSize: 72
BlinkTextColor: 0xFF6B35
BlinkEnableBg: true
BlinkBgColor: 0x000000
BlinkBgOpacity: 50
```

#### Direct Text Content (Recommended)
```
BlinkContent: "Thanks for following!"
BlinkTaskLength: 4
BlinkFontFamily: "Arial Black"
BlinkFontSize: 64
BlinkTextColor: 0xFFD700
BlinkFontStyle: "Bold"
```

#### Direct Text Content (Legacy BlinkVidPath)
```
BlinkVidPath: "Welcome to the stream!"
BlinkForceType: "Text"  // Required when using BlinkVidPath for text
BlinkTaskLength: 5
BlinkFontFamily: "Impact"
BlinkFontSize: 48
BlinkTextColor: 0xFFFFFF
BlinkFontStyle: "Bold"
```

## Usage Examples

### Media Sources

#### Basic Video Alert (Native Size)
```
BlinkContent: "C:\Alerts\follow-alert.mp4"
BlinkTaskLength: 3
BlinkPosX: 960         (positioned but keeps native dimensions)
BlinkPosY: 100
BlinkAudioMonitor: false
```

#### Basic Video Alert (Custom Size)
```
BlinkContent: "C:\Alerts\follow-alert.mp4"
BlinkTaskLength: 3
BlinkWidth: 400        (custom dimensions)
BlinkHeight: 200
BlinkPosX: 960
BlinkPosY: 100
BlinkAudioMonitor: false
```

#### Green Screen Effect with Audio
```
BlinkContent: "D:\Effects\explosion.mp4"
BlinkChroma: 0x00FF00
BlinkChromaSim: 400
BlinkChromaSmooth: 80
BlinkTaskLength: 8
BlinkAudioMonitor: true
BlinkVolume: 100
BlinkTracks: "all"
```

#### Video Without Chroma Key (Fullscreen)
```
BlinkContent: "D:\Overlays\transition.mp4"
BlinkChroma: -1        (no chroma key filtering)
BlinkTaskLength: 3
BlinkWidth: 1920       (fullscreen dimensions)
BlinkHeight: 1080
BlinkPosX: 0
BlinkPosY: 0
BlinkAudioMonitor: false
BlinkVolume: 50
BlinkTracks: "none"
```

#### Audio-Only Alert
```
BlinkContent: "D:\Sounds\notification.mp3"
BlinkTaskLength: 4
BlinkAudioMonitor: true
BlinkVolume: 80          (set playback volume)
BlinkTracks: "1"         (enable recording track 1 only)
BlinkWidth: 1            (no scaling needed for audio)
BlinkHeight: 1
```

### Image Sources

#### Donation Alert Badge (Native Size)
```
BlinkContent: "D:\Badges\donator.png"
BlinkTaskLength: 6
BlinkPosX: 1750        (positioned at native size)
BlinkPosY: 50
```

#### Donation Alert Badge (Custom Size)
```
BlinkContent: "D:\Badges\donator.png"
BlinkTaskLength: 6
BlinkWidth: 150        (custom badge size)
BlinkHeight: 150
BlinkPosX: 1750
BlinkPosY: 50
```

#### Overlay Image Without Background Removal (Fullscreen)
```
BlinkContent: "D:\Graphics\frame.png"
BlinkChroma: -1        (preserves original image transparency)
BlinkTaskLength: 10
BlinkWidth: 1920       (fullscreen overlay)
BlinkHeight: 1080
BlinkPosX: 0
BlinkPosY: 0
```

### Text Sources

#### Welcome Message from File
```
BlinkContent: "D:\Messages\welcome.txt"
BlinkTaskLength: 10
BlinkFontFamily: "Roboto"
BlinkFontSize: 64
BlinkTextColor: 0x00FF88
BlinkPosX: 960
BlinkPosY: 300
BlinkEnableBg: true
BlinkBgOpacity: 75
```

#### Configuration File Display
```
BlinkContent: "D:\Config\settings.json"
BlinkTaskLength: 15
BlinkFontFamily: "Consolas"
BlinkFontSize: 24
BlinkTextColor: 0x90EE90
BlinkEnableBg: true
BlinkBgColor: 0x2F4F4F
BlinkBgOpacity: 90
```

#### Dynamic Follower Notification (Recommended)
```
BlinkContent: "Thanks for the follow, %user%!"
BlinkTaskLength: 5
BlinkFontFamily: "Arial Black"
BlinkFontSize: 56
BlinkTextColor: 0xFFD700
BlinkFontStyle: "Bold"
BlinkPosX: 960
BlinkPosY: 200
```

#### Dynamic Follower Notification (Legacy)
```
BlinkVidPath: "Thanks for the follow, %user%!"
BlinkForceType: "Text"  // Required when using BlinkVidPath for text
BlinkTaskLength: 5
BlinkFontFamily: "Arial Black"
BlinkFontSize: 56
BlinkTextColor: 0xFFD700
BlinkFontStyle: "Bold"
BlinkPosX: 960
BlinkPosY: 200
```

#### Chat Message Display (Custom Size)
```
BlinkContent: "%message%"
BlinkTaskLength: 8
BlinkFontFamily: "Consolas"
BlinkFontSize: 32
BlinkTextColor: 0xFFFFFF
BlinkWidth: 800        (custom text area size)
BlinkHeight: 400
BlinkPosX: 100
BlinkPosY: 500
BlinkEnableBg: true
BlinkBgColor: 0x1E1E1E
BlinkBgOpacity: 85
```

#### Chat Message Display (Native Size)
```
BlinkContent: "%message%"
BlinkTaskLength: 8
BlinkFontFamily: "Consolas"
BlinkFontSize: 32
BlinkTextColor: 0xFFFFFF
BlinkPosX: 100         (positioned but native text size)
BlinkPosY: 500
BlinkEnableBg: true
BlinkBgColor: 0x1E1E1E
BlinkBgOpacity: 85
```

## How It Works

1. **Parameter Extraction**: Safely extracts and validates all configuration parameters
2. **Content Resolution**: Checks `BlinkVidPath` first for backward compatibility, then `BlinkContent`
3. **FontSettings Creation**: Creates a font configuration object for text sources
4. **Type Detection**: Intelligently determines source type from file extension or force override
5. **Smart File Handling**: If a file doesn't exist, automatically treats content as direct text
6. **Source Creation**: Creates appropriate OBS source (Media/Image/Text) in current scene
7. **Conditional Filtering**: Adds chroma key filtering only if `BlinkChroma >= 0`
8. **Audio Setup**: Configures audio monitoring for media sources only
9. **Smart Transform**: Applies custom scaling only when both width AND height are specified (non-zero)
10. **Async Cleanup**: Starts a timer to automatically remove the source
6. **Filter Application**: Adds chroma key filtering for media and image sources
7. **Audio Setup**: Configures audio monitoring for media sources only
8. **Transform Setup**: Sets position and size properties for all sources
9. **Async Cleanup**: Starts a timer to automatically remove the source

### Technical Flow

```mermaid
graph TD
    A[Execute Action] --> B[Extract Parameters]
    B --> C[Content Resolution: BlinkVidPath (legacy check) or BlinkContent]
    C --> D[Create FontSettings Object]
    D --> E{Force Type Set?}
    E -->|Yes| F[Use Force Type]
    E -->|No| G[Detect from Extension]
    F --> H{File-based Source?}
    G --> H
    H -->|Yes| I{File Exists?}
    H -->|No Text| J[Create Source]
    I -->|No| K[Fallback to Text Source]
    I -->|Yes| J
    K --> J
    J --> L{Source Type?}
    L -->|Media| M[Add Media Source]
    L -->|Image| N[Add Image Source]
    L -->|TextFile| O[Add Text Source from File]
    L -->|Text| P[Add Text Source Direct]
    M --> Q{Chroma >= 0?}
    N --> Q
    O --> R[Set Transform]
    P --> R
    Q -->|Yes| S[Add Chroma Filter]
    Q -->|No| T{Audio Monitor & Media?}
    S --> T
    T -->|Yes| U[Add Audio Monitor]
    T -->|No| V{Width & Height > 0?}
    U --> V
    V -->|Yes| W[Apply Custom Scaling + Position]
    V -->|No| X[Position Only - Native Size]
    W --> Y[Start Cleanup Timer]
    X --> Y
    Y --> Z[Source Active]
    Z --> AA[Timer Expires]
    AA --> BB[Remove Source]
```

## Code Structure

### `CPHInline` Class
- Main entry point for Streamer.bot
- Handles parameter parsing and validation with type safety
- Creates FontSettings object from font parameters
- Manages the `ObsMediaTask` lifecycle

### `ObsMediaTask` Class
- Encapsulates all OBS operations using modern architecture
- Manages multi-type source creation, configuration, and cleanup
- Handles asynchronous timing operations
- Uses enum-based dispatching for clean code organization

### `FontSettings` Class
- Data structure for font configuration with clean parameter passing
- Contains: Family, Size, Color, Style, BgColor, BgOpacity, EnableBg
- Compatible with .NET Framework versions used by Streamer.bot

### `SourceType` Enum
- Type-safe source type definitions: Media, Image, TextFile, Text
- Used for intelligent dispatching and type validation

### Key Methods
- `TryGetArg<T>()`: Safe parameter extraction with type conversion and defaults
- `DetermineSourceType()`: Intelligent source type detection from file extensions
- `AddSource()`: Smart dispatcher that routes to appropriate source creation method
- `AddMediaSource()`: Creates ffmpeg_source for video/audio files
- `AddImageSource()`: Creates image_source for static images
- `AddTextSource()`: Creates text_gdiplus_v3 for direct text content
- `AddTextSourceFromFile()`: Creates text_gdiplus_v3 from file content
- `AddFilter()`: Applies chroma key filtering (media/image sources only)
- `AddMonitor()`: Configures audio monitoring (media sources only)
- `SetTransform()`: Configures size and position for all sources
- `Start()`: Begins the timed cleanup process

## Troubleshooting

### Common Issues

**File Not Found Error**
```
Solution: Files now automatically fallback to text content
Note: Missing files are treated as text, not errors
Check: Logs will show "File not found" but continue execution
```

**Source Not Appearing**
```
Solution: Check OBS WebSocket connection and ensure scene exists
Verify: Current scene name and source creation logs
Check: For Text sources, file validation only applies to file-based sources
```

**Chroma Key Not Working**
```
Solution: Adjust similarity and smoothness values, or disable with -1
Try: Different chroma key color values (0x00FF00, 0xFF00FF, 0x0000FF)
Note: Use BlinkChroma: -1 to disable chroma key entirely
```

**No Transparency Effect**
```
Solution: Set BlinkChroma: -1 to disable chroma key
Alternative: Use proper chroma key colors (0x00FF00 for green screen)
Note: -1 preserves original image/video transparency
```

**Performance Issues**
```
Solution: Reduce video resolution or bitrate
Consider: Shorter task lengths for frequent use
```

### Debug Logging

The script provides comprehensive logging:
- Source type detection and validation
- Source creation confirmation with type information
- Font configuration details for text sources
- Task duration tracking and timer management
- Error reporting for failed operations
- File existence validation for file-based sources
- Audio monitoring status (media sources only)
- Cleanup completion notifications with source removal confirmation

## Advanced Configuration

### Dual Parameter Support
```csharp
// Recommended parameter (modern naming)
BlinkContent: "D:\Media\alert.mp4"

// Legacy parameter (backward compatibility only)
BlinkVidPath: "D:\Media\alert.mp4"

// System checks BlinkVidPath first for backward compatibility,
// then BlinkContent - but use BlinkContent for new implementations
```

### Custom Chroma Colors
```csharp
// Green screen (default)
BlinkChroma: 0x00FF00

// Blue screen  
BlinkChroma: 0x0000FF

// Magenta (excellent for graphics)
BlinkChroma: 0xFF00FF

// Custom purple
BlinkChroma: 0x800080

// Disable chroma key entirely
BlinkChroma: -1
```

### Font Styling Options
```csharp
// Bold text
BlinkFontStyle: "Bold"

// Italic text
BlinkFontStyle: "Italic"

// Bold and Italic
BlinkFontStyle: "Bold Italic"

// Custom font families
BlinkFontFamily: "Arial Black"
BlinkFontFamily: "Impact"
BlinkFontFamily: "Comic Sans MS"
BlinkFontFamily: "Roboto"
BlinkFontFamily: "Consolas"
```

### Text Background Configurations
```csharp
// Semi-transparent dark background
BlinkEnableBg: true
BlinkBgColor: 0x000000
BlinkBgOpacity: 75

// Solid colored background
BlinkEnableBg: true
BlinkBgColor: 0x1E90FF
BlinkBgOpacity: 100

// Subtle light background
BlinkEnableBg: true
BlinkBgColor: 0xF0F0F0
BlinkBgOpacity: 50
```

### Force Type Override Examples (Edge Cases Only)
```csharp
// RARE: Force text interpretation when using legacy BlinkVidPath
BlinkVidPath: "C:\Data\info.txt"
BlinkForceType: "Text"  // Treats the path as literal text, not a file to read

// BETTER: Use BlinkContent for direct text (no BlinkForceType needed)
BlinkContent: "C:\Data\info.txt"  // Automatically treated as text

// RARE: Force media source for unusual/custom extensions
BlinkContent: "D:\Videos\custom.xyz"
BlinkForceType: "Media"  // System can't detect .xyz, so force media type

// RARE: Force image source for non-standard formats  
BlinkContent: "E:\Images\special.data"
BlinkForceType: "Image"  // System can't detect .data, so force image type
```

**Note:** Use `BlinkContent` instead of `BlinkVidPath` to avoid needing `BlinkForceType` for text content.

### Multiple Simultaneous Sources
The GUID-based naming system allows multiple instances to run simultaneously without conflicts. Each source gets a unique 6-character identifier, enabling:
- Multiple alerts running concurrently
- Different source types in the same scene
- Overlapping timers without interference

## Contributing

This project is designed for Streamer.bot automation. Feel free to:
- Report issues or bugs
- Suggest feature improvements
- Share usage examples
- Contribute optimizations

## Support the Developer

This project was created by **ArnieTW**! If you find this tool useful for your streaming setup, consider showing your support:

🎮 **Follow on Twitch**: [twitch.tv/arnietw](https://twitch.tv/arnietw)  
☕ **Buy me a coffee**: [ko-fi.com/arnietw](https://ko-fi.com/arnietw)

Your follows and support help create more awesome tools for the streaming community! 💜

## License

This code is provided as-is for use with Streamer.bot. Modify and distribute according to your needs.

## Version History

- **v1.0**: Initial implementation with basic media source management
- **v2.0**: Added chroma key support, audio monitoring, and robust error handling
- **v3.0**: Multi-source type support (Media, Image, Text, TextFile)
- **v3.1**: Intelligent source type detection from file extensions
- **v3.2**: Force type override capability for edge cases
- **v3.3**: Upgraded to text_gdiplus_v3 for modern text rendering
- **v3.4**: Restricted audio monitoring to media sources only
- **v3.5**: Added comprehensive font customization parameters
- **v4.0**: Refactored to clean C# architecture with FontSettings class and enums
- **v4.1**: Added chroma key disable feature (use -1), dual parameter support (BlinkVidPath/BlinkContent), smart file fallback to text
 - **v4.1**: Added chroma key disable feature (use -1), dual parameter support (BlinkVidPath/BlinkContent), smart file fallback to text, initial sizing logic clarification
 - **v4.2**: Media audio enhancements: volume control (BlinkVolume), recording track mapping (BlinkTracks), expanded media detection to include common audio-only formats (.mp3, .wav, .ogg, .m4a, .aac, .flac, .opus)

---

*For Streamer.bot support and community resources, visit the official Streamer.bot Discord server.*