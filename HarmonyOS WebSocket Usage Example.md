# HarmonyOS Application Development: WebSocket Usage Example

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) represents a revolutionary step in the evolution of Huawei's distributed operating system. As someone who has been following its development closely, I can attest to the remarkable improvements in this version. The system's microkernel architecture not only enhances security but also provides unprecedented flexibility in cross-device collaboration. What excites me most about HarmonyOS 5 is its focus on developer experience - the new ArkTS language, enhanced UI components, and improved debugging tools have made development much more efficient. The distributed capabilities allow us to create truly seamless experiences across different devices, from smartphones to tablets, smart TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer based in China, with over three years of experience in the ecosystem. My journey with HarmonyOS began with its early versions, and I've witnessed its remarkable evolution. As a developer who has contributed to several open-source HarmonyOS projects, I've gained valuable insights into its architecture and best practices. I'm particularly passionate about real-time communication and distributed applications, which led me to explore WebSocket implementations in HarmonyOS 5. Through my blog and GitHub repositories, I've been sharing my experiences and helping other developers navigate the exciting world of HarmonyOS development.

## Preface
Recently, while developing HarmonyOS applications, I encountered a need for real-time communication. After some research, I found that HarmonyOS 5.0 provides more comprehensive WebSocket support, so I created a simple demo to test it. Below, I'll share my implementation process, hoping it helps others with similar needs.

## Development Environment
- DevEco Studio 4.0
- HarmonyOS SDK API 14 (HarmonyOS 5.0)
- Test Device: Huawei Mate 60 Pro

## Implementation Process

### 1. Import Required Packages
```typescript
import { webSocket } from '@kit.NetworkKit';
import { BusinessError } from '@kit.BasicServicesKit';
```

### 2. Create a Simple Component
Here I've created a simple component mainly for testing WebSocket's ping functionality. HarmonyOS 5.0's component system has seen significant improvements, making it more convenient to use.

```typescript
@Entry
@Component
struct WebSocketExample {
  @State pingResult: string = '';
  private ws: webSocket.WebSocket | undefined = undefined;
  private pingStartTime: number = 0;
  private pingCount: number = 0;
  private totalPingTime: number = 0;

  // Initialize when component is created
  aboutToAppear() {
    this.initWebSocket();
  }

  // Remember to close connection when component is destroyed
  aboutToDisappear() {
    this.closeWebSocket();
  }
}
```

### 3. Initialize WebSocket
HarmonyOS 5.0's WebSocket API has been significantly optimized with improved error handling. This code section might look long, but it's essentially just a few event listeners:

```typescript
private initWebSocket() {
  // Create instance
  this.ws = webSocket.createWebSocket();
  
  // Handle successful connection
  this.ws.on('open', (err: BusinessError, value: Object) => {
    if (err) {
      console.error('Connection failed:', JSON.stringify(err));
      this.pingResult = 'Connection failed';
      return;
    }
    console.log('Connection successful!');
  });

  // Handle received messages
  this.ws.on('message', (error: BusinessError, value: string | ArrayBuffer) => {
    if (error) {
      console.error('Message error:', JSON.stringify(error));
      return;
    }
    
    // If it's a pong message, calculate the delay
    if (value === 'pong') {
      const endTime = Date.now();
      const duration = endTime - this.pingStartTime;
      this.pingCount++;
      this.totalPingTime += duration;
      const avgPing = this.totalPingTime / this.pingCount;
      this.pingResult = `Delay: ${duration}ms (Average: ${avgPing.toFixed(2)}ms)`;
    }
  });

  // Error handling
  this.ws.on('error', (err: BusinessError) => {
    console.error('Error occurred:', JSON.stringify(err));
    this.pingResult = 'Connection error';
  });

  // Handle connection close
  this.ws.on('close', (err: BusinessError, value: webSocket.CloseResult) => {
    console.log(`Connection closed: ${value.code} - ${value.reason}`);
  });

  // Start connecting to server
  this.ws.connect('ws://your-server-address', {
    header: {
      'Content-Type': 'application/json'
    },
    // New configuration options in API 14
    protocols: ['my-protocol'],
    timeout: 10000, // 10 second timeout
    proxy: {
      host: '192.168.0.150',
      port: 8888,
      exclusionList: []
    }
  }, (err: BusinessError, value: boolean) => {
    if (err) {
      console.error('Connection failed:', JSON.stringify(err));
      this.pingResult = 'Connection failed';
    }
  });
}
```

### 4. Implement Ping Functionality
WebSocket message sending in HarmonyOS 5.0 is more stable, and this functionality is simple:

```typescript
private sendPing() {
  if (!this.ws) {
    this.pingResult = 'Not connected yet';
    return;
  }

  this.pingStartTime = Date.now();
  this.ws.send('ping', (err: BusinessError, value: boolean) => {
    if (err) {
      console.error('Send failed:', JSON.stringify(err));
      this.pingResult = 'Send failed';
    }
  });
}
```

### 5. Close Connection
Connection closing in API 14 is more reliable:

```typescript
private closeWebSocket() {
  if (this.ws) {
    this.ws.close((err: BusinessError) => {
      if (err) {
        console.error('Close failed:', JSON.stringify(err));
      }
    });
  }
}
```

### 6. Create a Simple Interface
HarmonyOS 5.0's UI system has seen significant improvements, making it more convenient to use:

```typescript
build() {
  Column() {
    Button('Test Latency')
      .onClick(() => this.sendPing())
    Text(this.pingResult)
      .fontSize(16)
      .margin({ top: 10 })
  }
  .width('100%')
  .height('100%')
  .justifyContent(FlexAlign.Center)
}
```

## Common Pitfalls
1. Initially forgot to close connections when components were destroyed, leading to memory leaks
2. Poor error handling causing application crashes
3. Connection timeout set too short, resulting in frequent connection failures
4. Forgot to implement reconnection logic, leading to poor experience during network instability
5. Didn't properly utilize new configuration options in API 14, causing unstable connections

## Summary
Although this demo is simple, it covers the main functionality of WebSocket. HarmonyOS 5.0's WebSocket API has seen many improvements, making it more stable to use. In actual projects, you might need to add:
- Automatic reconnection
- Heartbeat detection
- Message queue
- Offline message retransmission
and other features, depending on project requirements.

## References
- [HarmonyOS 5.0 WebSocket API Documentation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)
- [WebSocket Protocol Specification](https://tools.ietf.org/html/rfc6455)

## Postscript
While writing this article, I noticed that HarmonyOS 5.0's documentation has been significantly updated, and the APIs have become more user-friendly. Particularly the WebSocket module, which has added many practical features. I hope this article helps those learning HarmonyOS development. If you have any questions, feel free to discuss in the comments section.