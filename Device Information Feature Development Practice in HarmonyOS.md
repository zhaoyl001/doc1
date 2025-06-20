# Device Information Feature Development Practice in HarmonyOS

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) represents a revolutionary step in the evolution of Huawei's distributed operating system. As someone who has been following its development closely, I can attest to the remarkable improvements in this version. The system's microkernel architecture not only enhances security but also provides unprecedented flexibility in cross-device collaboration. What excites me most about HarmonyOS 5 is its focus on developer experience - the new ArkTS language, enhanced UI components, and improved debugging tools have made development much more efficient. The distributed capabilities allow us to create truly seamless experiences across different devices, from smartphones to tablets, smart TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer based in China, with over three years of experience in the ecosystem. My journey with HarmonyOS began with its early versions, and I've witnessed its remarkable evolution. As a developer who has contributed to several open-source HarmonyOS projects, I've gained valuable insights into its architecture and best practices. I'm particularly passionate about device information and system integration, which led me to develop this comprehensive device information module. Through my blog and GitHub repositories, I've been sharing my experiences and helping other developers navigate the exciting world of HarmonyOS development.

## Preface
During the development of the HarmonyOS Developer Toolbox, we frequently needed to debug device information-related features. To facilitate development and debugging, we developed a device information module. This module not only includes basic device information but also integrates detailed information about networks, screens, cameras, and sensors, providing convenience for development and debugging.

## I. Feature Overview

The device information feature mainly provides the following information:

### 1.1 Basic Device Information
- Device Type
- Manufacturer
- Brand
- Product Name
- Product Model
- Hardware Model
- Product Series
- Software Model

### 1.2 System Information
- System Version
- Release Type
- Display Version
- Build Type
- System Name
- System Version
- API Version
- SDK API Version

### 1.3 Build Information
- Build Type
- Build User
- Build Host
- Build Time

### 1.4 Hardware Information
- Serial Number
- Boot Version
- Security Patch
- ABI List

### 1.5 Network Information
- Network Type (WiFi/Mobile)
- IP Address
- Gateway
- Subnet Mask
- WiFi Signal Strength
- Frequency Band Information
- IPv6 Information

### 1.6 Screen Information
- Resolution
- Refresh Rate
- Screen Density
- HDR Support
- Color Space

### 1.7 Camera Information
- Camera Type (Front/Rear)
- Camera Parameters
- Supported Resolutions
- Frame Rate Range

### 1.8 Sensor Information
- Sensor Type
- Vendor Information
- Technical Parameters
- Sampling Period

## II. Technical Implementation

### 2.1 Using Device Information APIs

Using multiple API modules provided by HarmonyOS to obtain device information:

```typescript
import deviceInfo from '@ohos.deviceInfo';
import display from '@ohos.display';
import { connection } from '@kit.NetworkKit';
import { wifiManager } from '@kit.ConnectivityKit';
import { camera } from '@kit.CameraKit';
import { sensor } from '@kit.SensorServiceKit';
```

### 2.2 Data Model Design

To manage various device information, we designed corresponding data models:

```typescript
// Basic device information model
interface DeviceInfoData {
  // Basic information
  deviceType: string;
  manufacture: string;
  brand: string;
  // ... other basic information fields
}

// Network information model
interface NetworkInfoData {
  networkType: string;
  isConnected: boolean;
  ipAddress: string;
  // ... other network information fields
}

// Camera information model
interface CameraInfo {
  cameraId: string;
  cameraPosition: string;
  cameraType: string;
  // ... other camera information fields
}
```

### 2.3 UI Component Design

Using ArkUI's component system to build the interface, mainly including:

1. Information Display Components
   - Title and content layout
   - Copy functionality
   - Error prompts

2. Category Display Components
   - Card-style layout
   - Group display
   - Scroll support

3. Navigation Components
   - Top navigation bar
   - Tab switching
   - Back button

### 2.4 Core Function Implementation

1. Device Information Retrieval
```typescript
// Get basic device information
async getDeviceInfo() {
  try {
    const deviceInfo = {
      deviceType: deviceInfo.deviceType,
      manufacture: deviceInfo.manufacture,
      brand: deviceInfo.brand,
      // ... other information
    };
    return deviceInfo;
  } catch (error) {
    console.error('Failed to get device information:', error);
    return null;
  }
}
```

2. Network Information Monitoring
```typescript
// Network status monitoring
private networkCallback = {
  onAvailable: (netHandle) => {
    this.updateNetworkInfo(netHandle);
  },
  onLost: () => {
    this.networkData.isConnected = false;
  }
};

// Register network monitoring
private registerNetworkCallback() {
  connection.on('netAvailable', this.networkCallback);
  connection.on('netLost', this.networkCallback);
}
```

3. Camera Information Retrieval
```typescript
// Get camera list
async getCameraList() {
  try {
    const cameraManager = await camera.getCameraManager();
    const cameraList = await cameraManager.getCameras();
    return cameraList.map(camera => ({
      id: camera.cameraId,
      position: camera.position,
      // ... other information
    }));
  } catch (error) {
    console.error('Failed to get camera list:', error);
    return [];
  }
}

// Get detailed camera information
async getCameraDetail(cameraId: string) {
  try {
    const cameraManager = await camera.getCameraManager();
    const cameraDevice = await cameraManager.getCamera(cameraId);
    
    // Get camera capabilities
    const capabilities = await cameraManager.getSupportedOutputCapability(
      cameraDevice,
      camera.SceneMode.NORMAL_PHOTO
    );
    
    return {
      id: cameraDevice.cameraId,
      position: this.getCameraPosition(cameraDevice.cameraPosition),
      type: this.getCameraType(cameraDevice.cameraType),
      // Preview resolution
      previewSizes: capabilities.previewProfiles.map(profile => 
        `${profile.size.width}×${profile.size.height}`
      ),
      // Photo resolution
      photoSizes: capabilities.photoProfiles.map(profile => 
        `${profile.size.width}×${profile.size.height}`
      ),
      // Video resolution
      videoSizes: capabilities.videoProfiles.map(profile => 
        `${profile.size.width}×${profile.size.height} @${profile.frameRateRange.min}-${profile.frameRateRange.max}fps`
      )
    };
  } catch (error) {
    console.error('Failed to get camera details:', error);
    return null;
  }
}

// Get camera position description
private getCameraPosition(position: number): string {
  switch (position) {
    case camera.Position.POSITION_FRONT:
      return 'Front';
    case camera.Position.POSITION_BACK:
      return 'Back';
    default:
      return 'Unknown';
  }
}

// Get camera type description
private getCameraType(type: number): string {
  switch (type) {
    case camera.CameraType.CAMERA_TYPE_WIDE_ANGLE:
      return 'Wide Angle';
    case camera.CameraType.CAMERA_TYPE_ULTRA_WIDE:
      return 'Ultra Wide';
    case camera.CameraType.CAMERA_TYPE_TELEPHOTO:
      return 'Telephoto';
    default:
      return 'Unknown';
  }
}
```

### 2.5 Permission Management Example

```typescript
// Permission check
async checkPermission(permission: string): Promise<boolean> {
  try {
    const result = await bundleManager.checkPermission({
      bundleName: this.bundleName,
      permission: permission
    });
    return result === bundleManager.GrantStatus.PERMISSION_GRANTED;
  } catch (error) {
    console.error(`Permission check failed: ${error}`);
    return false;
  }
}

// Permission request
async requestPermission(permission: string): Promise<boolean> {
  try {
    const result = await bundleManager.requestPermission({
      bundleName: this.bundleName,
      permission: permission
    });
    return result === bundleManager.GrantStatus.PERMISSION_GRANTED;
  } catch (error) {
    console.error(`Permission request failed: ${error}`);
    return false;
  }
}

// Permission usage example
async getCameraInfo() {
  const hasPermission = await this.checkPermission('ohos.permission.CAMERA');
  if (!hasPermission) {
    const granted = await this.requestPermission('ohos.permission.CAMERA');
    if (!granted) {
      promptAction.showToast({ message: 'Camera permission required to get camera information' });
      return;
    }
  }
  // ... specific implementation of getting camera information
}
```

## III. Feature Characteristics

### 3.1 Information Display
- Categorized display of device information
- Information copy functionality
- Real-time network status updates
- Dark mode support

### 3.2 Performance Optimization
- Singleton pattern for device information management
- Controlled information update frequency
- Optimized UI rendering performance
- Appropriate cache usage

### 3.3 User Experience
- Favorite functionality
- Clear information categorization
- Manual refresh support
- Friendly error prompts

## IV. Development Experience

### 4.1 Permission Management
- Camera information requires camera permission
- Network information requires network permission
- Sensor information requires sensor permission
- Proper handling of permission requests and denials

### 4.2 Error Handling
- Network request exception handling
- Device information retrieval failure handling
- Permission request failure handling
- Friendly error prompts

### 4.3 Performance Optimization
- Controlled update frequency
- Optimized UI rendering
- Appropriate cache usage
- Timely resource release

## V. Problems Encountered

### 5.1 Permission Issues
- Problem: Some device information requires special permissions
- Solution: Added permission request and check mechanisms with friendly prompts

### 5.2 Network Information Updates
- Problem: Frequent updates affecting performance
- Solution: Set update intervals to balance real-time performance and efficiency

### 5.3 Device Compatibility
- Problem: Inconsistent information formats across different devices
- Solution: Added data format conversion and compatibility handling

### 5.4 Memory Management
- Problem: Long-term operation may cause memory leaks
- Solution: Timely release of listeners and timers

## VI. Important Notes

1. Permission Management
   - Reasonable permission requests
   - Handle permission denials
   - Provide friendly prompts

2. Performance Optimization
   - Control update frequency
   - Optimize UI rendering
   - Release resources promptly

3. User Experience
   - Provide friendly error prompts
   - Support information copying
   - Keep interface clean

4. Security
   - Protect sensitive information
   - Handle permissions properly
   - Avoid information leakage

## VII. Summary

The device information feature provides developers with a comprehensive device information viewing tool, including:

- Basic device information
- System information
- Network information
- Screen information
- Camera information
- Sensor information

Through this feature, developers can:

- Quickly obtain device information
- Debug application performance
- Optimize application experience
- Solve compatibility issues

## VIII. Reference Resources

- [HarmonyOS Application Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

---

> Author: In the World of Development
> Email: 1743914721@qq.com
> Copyright Notice: This article is an original work by the CSDN blogger, please include the original source link and this statement when reprinting. 