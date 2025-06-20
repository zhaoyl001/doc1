# Network Speed Test Tool Development Notes in HarmonyOS

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) represents a revolutionary step in the evolution of Huawei's distributed operating system. As someone who has been following its development closely, I can attest to the remarkable improvements in this version. The system's microkernel architecture not only enhances security but also provides unprecedented flexibility in cross-device collaboration. What excites me most about HarmonyOS 5 is its focus on developer experience - the new ArkTS language, enhanced UI components, and improved debugging tools have made development much more efficient. The distributed capabilities allow us to create truly seamless experiences across different devices, from smartphones to tablets, smart TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer based in China, with over three years of experience in the ecosystem. My journey with HarmonyOS began with its early versions, and I've witnessed its remarkable evolution. As a developer who has contributed to several open-source HarmonyOS projects, I've gained valuable insights into its architecture and best practices. I'm particularly passionate about network tools and performance testing, which led me to develop this comprehensive network speed test module. Through my blog and GitHub repositories, I've been sharing my experiences and helping other developers navigate the exciting world of HarmonyOS development.

## Preface
Recently, while developing the HarmonyOS Developer Toolbox, we needed to add a network speed test feature. This feature is mainly used to measure network speed, including download speed, upload speed, and network latency. During actual development, we found that HarmonyOS's support for network speed testing was not very comprehensive, so we had to write our own speed test logic. After several debugging attempts, we finally developed a usable version.

## I. Feature Description

### 1.1 Main Features
- Download speed testing
- Upload speed testing
- Network latency testing
- Customizable test servers
- Automatic unit conversion (Kbps/Mbps/Gbps)

### 1.2 Interface Features
- Real-time speed display
- Test progress display
- Custom server settings
- Favorites support
- Usage instructions

## II. Implementation Process

### 2.1 Speed Test Principle

The basic idea of speed testing is to measure network speed by downloading and uploading files. In HarmonyOS, we mainly use HTTP requests to implement this:

1. Download Speed Test
   - Download a test file from the server (we use 10MB)
   - Record start time
   - Calculate downloaded data amount in real-time
   - Speed = Data amount/Time
   - Take average of multiple tests for accuracy

2. Upload Speed Test
   - Generate test data (also 10MB)
   - Record start time
   - Calculate uploaded data amount in real-time
   - Speed = Data amount/Time
   - Multiple tests for accuracy

3. Latency Test
   - Send HTTP HEAD request
   - Record send time
   - Record response time
   - Latency = Response time - Send time
   - Take minimum of multiple tests

4. Unit Conversion
   - 1 Mbps = 1024 Kbps
   - 1 Gbps = 1024 Mbps
   - Automatically select appropriate unit based on speed

5. Server Selection
   - Maintain server list
   - Test latency for each server
   - Select server with lowest latency
   - Support custom server settings

### 2.2 Code Implementation

```typescript
@Entry
@Component
struct SpeedTest {
  @State downloadSpeed: number = 0;      // Download speed
  @State uploadSpeed: number = 0;        // Upload speed
  @State isTesting: boolean = false;     // Testing status
  @State currentTest: string = '';       // Current test type
  @State progress: number = 0;           // Progress
  @State testServer: string = '';        // Test server
  @State ping: number = 0;               // Latency

  // Start test
  private async startTest(): Promise<void> {
    try {
      this.resetState();
      this.isTesting = true;
      this.startProgressAnimation();

      // Select server
      this.currentTest = 'Selecting server...';
      this.testServer = await NetworkService.selectSpeedTestServer();
      
      // Test latency
      this.currentTest = 'Testing latency...';
      this.ping = await NetworkService.testPing(this.testServer);

      // Test download
      this.currentTest = 'Testing download speed...';
      this.downloadSpeed = await NetworkService.testDownloadSpeed(
        this.testServer,
        (progress) => {
          this.progress = progress;
        }
      );

      // Test upload
      this.currentTest = 'Testing upload speed...';
      this.uploadSpeed = await NetworkService.testUploadSpeed(
        this.testServer,
        (progress) => {
          this.progress = progress;
        }
      );

      this.currentTest = 'Test complete';
    } catch (error) {
      this.isError = true;
      this.currentTest = 'Test failed';
    } finally {
      this.isTesting = false;
      this.stopProgressAnimation();
    }
  }

  // Format speed display
  private formatSpeedValue(speed: number): string {
    if (speed === 0) return '0';
    if (speed >= 1000) return (speed / 1000).toFixed(1);
    if (speed >= 1) return speed.toFixed(1);
    return (speed * 1000).toFixed(0);
  }

  private formatSpeedUnit(speed: number): string {
    if (speed >= 1000) return 'Gbps';
    if (speed >= 1) return 'Mbps';
    return 'Kbps';
  }
}
```

## III. Development Experience

### 3.1 Problems Encountered

1. Speed Test Accuracy
   - Problem: Large speed fluctuations
   - Solution: Take average of multiple tests

2. Server Issues
   - Problem: Different servers show varying results
   - Solution: Automatically select best server

3. Progress Display
   - Problem: Progress updates not timely
   - Solution: Added animation effects

4. Memory Issues
   - Problem: High memory usage
   - Solution: Release resources promptly

### 3.2 Optimization Suggestions

1. Speed Test Optimization
   - Multiple tests
   - Adjust test file size
   - Support resumable downloads

2. Performance Optimization
   - Optimize UI
   - Reduce memory usage
   - Release resources promptly

3. User Experience
   - Add test history
   - Support result sharing
   - Optimize animations

4. Security
   - Validate server addresses
   - Limit test frequency
   - Protect user privacy

## IV. Summary

This speed test tool has all the basic features and can:

- Test network speed
- Check network issues
- Optimize network settings
- Evaluate network quality

## V. Reference Resources

- [HarmonyOS Application Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## Welcome to Experience

This speed test tool has been integrated into the HarmonyOS Developer Toolbox. Welcome to download and experience it!

[HarmonyOS Developer Toolbox](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> Author: In the World of Development
> Email: 1743914721@qq.com
> Copyright Notice: This article is an original work by the blogger, please include the original source link and this statement when reprinting. 