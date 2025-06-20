# IPv4 Subnet Calculator Development Notes in HarmonyOS

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) represents a revolutionary step in the evolution of Huawei's distributed operating system. As someone who has been following its development closely, I can attest to the remarkable improvements in this version. The system's microkernel architecture not only enhances security but also provides unprecedented flexibility in cross-device collaboration. What excites me most about HarmonyOS 5 is its focus on developer experience - the new ArkTS language, enhanced UI components, and improved debugging tools have made development much more efficient. The distributed capabilities allow us to create truly seamless experiences across different devices, from smartphones to tablets, smart TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer based in China, with over three years of experience in the ecosystem. My journey with HarmonyOS began with its early versions, and I've witnessed its remarkable evolution. As a developer who has contributed to several open-source HarmonyOS projects, I've gained valuable insights into its architecture and best practices. I'm particularly passionate about network tools and subnet management, which led me to develop this comprehensive IPv4 subnet calculator module. Through my blog and GitHub repositories, I've been sharing my experiences and helping other developers navigate the exciting world of HarmonyOS development.

## Basic Knowledge

Let's first understand IP addresses and subnets:

1. What is an IP Address
   - It's an address assigned to network devices
   - For example, your computer's IP might be 192.168.1.100
   - Each number ranges from 0-255
   - Total of 4 numbers, separated by dots

2. What is a Subnet Mask
   - Used to distinguish network and host portions
   - For example, 255.255.255.0
   - 1s represent network portion
   - 0s represent host portion

3. What is CIDR
   - Shorthand for subnet mask
   - For example, /24 means 255.255.255.0
   - Number represents count of 1s
   - Range is 0-32

4. How to Calculate Network Address
   - Perform AND operation between IP and subnet mask
   - For example, 192.168.1.100/24
   - Network address is 192.168.1.0

5. What is Broadcast Address
   - Used to send messages to entire subnet
   - Last octet of network address becomes 255
   - For example, 192.168.1.255

## Preface
Recently, while developing the HarmonyOS Developer Toolbox, we wanted to add a subnet calculator feature. This feature is mainly used to calculate subnet information for IP addresses, such as network address, broadcast address, and available host count. Initially, we thought it would be simple, but we found that we needed to handle various edge cases, and it took several debugging attempts to get it working.

## I. Feature Description

### 1.1 Main Features
- Calculate network address
- Calculate broadcast address
- Calculate available hosts
- Calculate subnet mask
- Calculate wildcard mask
- Display IP class
- Support CIDR format

### 1.2 Interface Features
- IP address input field
- One-click copy results
- Display detailed calculations
- Favorites support
- Usage instructions

## II. Implementation Process

### 2.1 Calculation Principle

Subnet calculation is essentially bitwise operations:

1. Network Address Calculation
   - AND operation between IP and subnet mask
   - For example, 192.168.1.1/24
   - Network address is 192.168.1.0

2. Broadcast Address Calculation
   - OR operation between network address and wildcard mask
   - For example, 192.168.1.0/24
   - Broadcast address is 192.168.1.255

3. Available Host Count Calculation
   - 2^(32-prefix length) - 2
   - For example, /24 is 2^8 - 2 = 254

4. Subnet Mask Calculation
   - Generated based on prefix length
   - For example, /24 is 255.255.255.0

5. IP Class Determination
   - Class A: 1-126
   - Class B: 128-191
   - Class C: 192-223
   - Class D: 224-239
   - Class E: 240-255

### 2.2 Code Implementation

```typescript
@Entry
@Component
struct Ipv4subnetcalculator {
  @State ipAddress: string = '';           // IP to calculate
  @State networkMask: string = '';         // Network mask
  @State networkAddress: string = '';      // Network address
  @State subnetMask: string = '';          // Subnet mask
  @State subnetMaskBinary: string = '';    // Subnet mask binary
  @State subnetMaskCIDR: string = '';      // Subnet mask CIDR
  @State wildcardMask: string = '';        // Wildcard mask
  @State totalHosts: number = 0;           // Available hosts
  @State firstHost: string = '';           // First address
  @State lastHost: string = '';            // Last address
  @State broadcastAddress: string = '';    // Broadcast address
  @State ipClass: string = '';             // IP class

  // Calculate subnet information
  private calculateSubnet() {
    try {
      // Preprocess input
      const processedInput = this.preprocessIPAddress(this.ipAddress);
      
      // Parse input
      let ip = processedInput;
      let cidr = '24'; // Default subnet mask
      if (processedInput.includes('/')) {
        const parts = processedInput.split('/');
        ip = parts[0];
        cidr = parts[1];
      }

      // Validate IP address format
      if (!this.isValidIPAddress(ip)) {
        throw new Error('Invalid IP address format');
      }

      const prefix = parseInt(cidr);
      if (isNaN(prefix) || prefix < 0 || prefix > 32) {
        throw new Error('Invalid CIDR prefix (0-32)');
      }

      // Calculate subnet mask
      const subnetMaskNum = ~((1 << (32 - prefix)) - 1);
      const subnetMaskParts = [
        (subnetMaskNum >>> 24) & 255,
        (subnetMaskNum >>> 16) & 255,
        (subnetMaskNum >>> 8) & 255,
        subnetMaskNum & 255
      ];
      this.subnetMask = subnetMaskParts.join('.');
      this.subnetMaskBinary = subnetMaskParts.map(n => 
        n.toString(2).padStart(8, '0')).join('.');
      this.subnetMaskCIDR = '/' + prefix;

      // Calculate wildcard mask
      const wildcardParts = subnetMaskParts.map(n => 255 - n);
      this.wildcardMask = wildcardParts.join('.');

      // Calculate network address
      const ipParts = ip.split('.').map(n => parseInt(n));
      const networkParts = ipParts.map((n, i) => n & subnetMaskParts[i]);
      this.networkAddress = networkParts.join('.');
      this.networkMask = this.networkAddress + this.subnetMaskCIDR;

      // Calculate broadcast address
      const broadcastParts = networkParts.map((n, i) => n | wildcardParts[i]);
      this.broadcastAddress = broadcastParts.join('.');

      // Calculate available host range
      const firstHostParts = [...networkParts];
      firstHostParts[3] += 1;
      this.firstHost = firstHostParts.join('.');

      const lastHostParts = [...broadcastParts];
      lastHostParts[3] -= 1;
      this.lastHost = lastHostParts.join('.');

      // Calculate available host count
      this.totalHosts = Math.pow(2, 32 - prefix) - 2;

      // Determine IP class
      const firstOctet = ipParts[0];
      if (firstOctet < 128) this.ipClass = 'A';
      else if (firstOctet < 192) this.ipClass = 'B';
      else if (firstOctet < 224) this.ipClass = 'C';
      else if (firstOctet < 240) this.ipClass = 'D';
      else this.ipClass = 'E';

    } catch (err) {
      console.error('Calculation failed:', err);
      throw err;
    }
  }
}
```

## III. Development Experience

### 3.1 Problems Encountered

1. IP Format Validation
   - Problem: Need to support various IP formats
   - Solution: Wrote a preprocessing function

2. Edge Cases
   - Problem: Special IP address handling
   - Solution: Added multiple checks

3. Performance Issues
   - Problem: Too frequent calculations
   - Solution: Added debouncing

4. Display Issues
   - Problem: Results too long to display
   - Solution: Optimized layout

### 3.2 Optimization Suggestions

1. Feature Optimization
   - Support more formats
   - Add calculation history
   - Support batch calculations

2. Performance Optimization
   - Optimize calculation speed
   - Reduce memory usage
   - Release resources promptly

3. User Experience
   - Add calculation history
   - Support result sharing
   - Optimize animations

4. Security
   - Validate input format
   - Limit calculation frequency
   - Protect user privacy

## IV. Summary

This subnet calculator has all the basic features and can:

- Calculate subnet information
- Plan network addresses
- Troubleshoot network issues
- Learn about networking

## V. Reference Resources

- [HarmonyOS Application Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## Welcome to Experience

This subnet calculator has been integrated into the HarmonyOS Developer Toolbox. Welcome to download and experience it!

[HarmonyOS Developer Toolbox](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> Author: In the World of Development
> Email: 1743914721@qq.com
> Copyright Notice: This article is an original work by the CSDN blogger, please include the original source link and this statement when reprinting. 