# Port Scanning Tool Development Practice in HarmonyOS Network Tools

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) represents a revolutionary step in the evolution of Huawei's distributed operating system. As someone who has been following its development closely, I can attest to the remarkable improvements in this version. The system's microkernel architecture not only enhances security but also provides unprecedented flexibility in cross-device collaboration. What excites me most about HarmonyOS 5 is its focus on developer experience - the new ArkTS language, enhanced UI components, and improved debugging tools have made development much more efficient. The distributed capabilities allow us to create truly seamless experiences across different devices, from smartphones to tablets, smart TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer based in China, with over three years of experience in the ecosystem. My journey with HarmonyOS began with its early versions, and I've witnessed its remarkable evolution. As a developer who has contributed to several open-source HarmonyOS projects, I've gained valuable insights into its architecture and best practices. I'm particularly passionate about network tools and security testing, which led me to develop this comprehensive port scanning module. Through my blog and GitHub repositories, I've been sharing my experiences and helping other developers navigate the exciting world of HarmonyOS development.

## Preface
Recently, while developing the HarmonyOS Developer Toolbox, we needed to implement a port scanning feature. This feature helps developers quickly detect open ports on target hosts, which is very useful for network debugging and security testing. After practice, we implemented this feature using HTTP protocol. Although it differs from traditional port scanning tools, it basically meets the requirements for development and debugging.

## I. Feature Overview

### 1.1 Basic Features
- Support for IP address and domain name scanning
- Customizable port range
- Real-time scan progress display
- Support for batch port scanning
- Beautiful result display

### 1.2 Interface Features
- Target address input
- Port range settings
- Scan progress display
- Result display
- Favorite functionality

## II. Technical Implementation

### 2.1 Implementation Principle

The port scanning implementation principle is relatively simple:
1. Attempt to connect to specified ports on target host through HTTP requests
2. If connection succeeds, port is open
3. If connection fails or times out, port is closed

### 2.2 Common Port Knowledge

When scanning ports, understanding the purpose of common ports is helpful:

1. Common Service Ports
   - 80/443: HTTP/HTTPS services
   - 21: FTP service
   - 22: SSH service
   - 23: Telnet service
   - 25: SMTP mail service
   - 53: DNS service
   - 3306: MySQL database
   - 3389: Remote desktop

2. Port Classification
   - 0-1023: System ports, require administrator privileges
   - 1024-49151: User ports, used by regular applications
   - 49152-65535: Dynamic ports, for temporary use

3. Scanning Recommendations
   - Recommend scanning common ports first
   - Avoid scanning system ports
   - Pay attention to scan frequency
   - Comply with relevant regulations

### 2.3 Core Code Implementation

```typescript
@Entry
@Component
struct PortScanner {
  @State host: string = '';              // Target host address
  @State portRange: string = '';         // Port range
  @State result: string = '';            // Scan result
  @State isScanning: boolean = false;    // Scan status
  @State progress: number = 0;           // Scan progress

  // Start scanning
  private async startScan(): Promise<void> {
    try {
      this.isScanning = true;
      this.progress = 0;
      this.result = 'Scanning in progress...\n';

      // Parse port range
      const range = this.portRange.split('-');
      const startPort = parseInt(range[0]);
      const endPort = parseInt(range[1]);

      // Execute port scan
      const openPorts = await NetworkService.scanPorts(
        this.host.trim(),
        startPort,
        endPort,
        (current, total) => {
          this.progress = Math.floor((current / total) * 100);
        }
      );

      // Process scan results
      if (openPorts.length > 0) {
        this.result = `Scan complete!\n\nFound ${openPorts.length} open ports:\n\n`;
        openPorts.forEach(port => {
          this.result += `📌 Port ${port} is open\n`;
        });
      } else {
        this.result = 'Scan complete! No open ports found.';
      }
    } catch (error) {
      this.result = `Scan failed: ${error?.message || 'Unknown error'}`;
    } finally {
      this.isScanning = false;
      this.progress = 100;
    }
  }
}

// Network service
export class NetworkService {
  // Scan ports
  static async scanPorts(
    host: string,
    startPort: number,
    endPort: number,
    progressCallback?: (current: number, total: number) => void
  ): Promise<number[]> {
    const openPorts: number[] = [];
    const total = endPort - startPort + 1;
    let current = 0;

    for (let port = startPort; port <= endPort; port++) {
      try {
        const httpRequest = http.createHttp();
        const response = await httpRequest.request(
          `http://${host}:${port}`,
          {
            method: http.RequestMethod.GET,
            connectTimeout: 1000,
            readTimeout: 1000
          }
        );
        
        if (response.responseCode >= 200 && response.responseCode < 400) {
          openPorts.push(port);
        }
      } catch (error) {
        // Connection failed, port may be closed
      } finally {
        current++;
        progressCallback?.(current, total);
      }
    }

    return openPorts;
  }
}
```

## III. Development Experience

### 3.1 Problems Encountered

1. Scan Efficiency
   - Problem: Slow scanning speed
   - Solution: Set reasonable timeout period

2. Input Validation
   - Problem: Invalid port range
   - Solution: Add format validation

3. Progress Display
   - Problem: Progress updates not timely
   - Solution: Use callback function

4. Resource Management
   - Problem: Memory usage
   - Solution: Release resources promptly

### 3.2 Optimization Suggestions

1. Input Validation
   - Check IP format
   - Validate port range
   - Handle special characters

2. Performance Optimization
   - Control scan frequency
   - Optimize UI rendering
   - Release resources promptly

3. User Experience
   - Display scan progress
   - Support cancel operation
   - Save scan history

4. Security
   - Limit scan range
   - Control scan frequency
   - Protect user privacy

## IV. Summary

The port scanning tool implements basic port detection functionality. Although implemented using HTTP protocol, it basically meets the requirements for development and debugging. Through this tool, you can:

- Detect port status
- Diagnose network issues
- Perform security testing
- Optimize network configuration

## V. Reference Resources

- [HarmonyOS Application Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## Welcome to Experience

The port scanning tool introduced in this article has been integrated into the HarmonyOS Developer Toolbox. Welcome to download and experience more features!

[HarmonyOS Developer Toolbox](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> Author: In the World of Development
> Email: 1743914721@qq.com
> Copyright Notice: This article is an original work by the CSDN blogger, please include the original source link and this statement when reprinting. 