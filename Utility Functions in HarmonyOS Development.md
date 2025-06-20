# Utility Functions in HarmonyOS Development

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) represents a revolutionary step in the evolution of Huawei's distributed operating system. As someone who has been following its development closely, I can attest to the remarkable improvements in this version. The system's microkernel architecture not only enhances security but also provides unprecedented flexibility in cross-device collaboration. What excites me most about HarmonyOS 5 is its focus on developer experience - the new ArkTS language, enhanced UI components, and improved debugging tools have made development much more efficient. The distributed capabilities allow us to create truly seamless experiences across different devices, from smartphones to tablets, smart TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer based in China, with over three years of experience in the ecosystem. My journey with HarmonyOS began with its early versions, and I've witnessed its remarkable evolution. As a developer who has contributed to several open-source HarmonyOS projects, I've gained valuable insights into its architecture and best practices. I'm particularly passionate about utility functions and performance optimization, which led me to explore the util package in depth. Through my blog and GitHub repositories, I've been sharing my experiences and helping other developers navigate the exciting world of HarmonyOS development.

## Preface
Recently, while working on HarmonyOS development, I found the util package quite useful, containing many practical functions. Today, I'll share some of the most commonly used ones, hoping it helps everyone.

## Development Environment
- DevEco Studio 4.0
- HarmonyOS SDK API 14 (HarmonyOS 5.0)
- Test Device: Huawei Mate 60 Pro

## Common Utility Functions

### 1. Import Package
```typescript
import { util } from '@kit.ArkTS';
```

### 2. String Processing
```typescript
// String formatting, most commonly used
let name = 'Zhang San';
let age = 25;
let str = util.format('My name is %s, I am %d years old', name, age);
console.info(str);  // Output: My name is Zhang San, I am 25 years old

// Object to string, frequently used in debugging
const person = {
  name: 'Zhang San',
  age: 25,
  address: {
    city: 'Beijing',
    country: 'China'
  }
};
console.info(util.format('Personal Info: %j', person));
// Output: Personal Info: {"name":"Zhang San","age":25,"address":{"city":"Beijing","country":"China"}}

// Number formatting, convenient for decimal handling
const pi = 3.141592653;
console.info(util.format('Value of π: %f', pi));  // Output: Value of π: 3.141592653
```

### 3. Error Handling
```typescript
// Get error message, essential for debugging
let errnum = -1;
let result = util.errnoToString(errnum);
console.info("Error message: " + result);  // Output: Error message: operation not permitted

// Common error codes, recommended to bookmark
const errorCodes = {
  '-1': 'operation not permitted',
  '-2': 'no such file or directory',
  '-3': 'no such process',
  '-4': 'interrupted system call',
  '-5': 'i/o error',
  '-11': 'resource temporarily unavailable',
  '-12': 'not enough memory',
  '-13': 'permission denied',
  '-100': 'network is down'
};
```

### 4. Asynchronous Processing
```typescript
// Callback to Promise, essential for legacy code refactoring
async function fetchData() {
  return 'hello world';
}

let cb = util.callbackWrapper(fetchData);
cb((err: Object, ret: string) => {
  if (err) throw new Error;
  console.info(ret);  // Output: hello world
});

// Promisify, makes coding more enjoyable
const addCall = util.promisify(util.callbackWrapper(fetchData));
(async () => {
  try {
    let res: string = await addCall();
    console.info(res);  // Output: hello world
  } catch (err) {
    console.info(err);
  }
})();
```

### 5. UUID Generation
```typescript
// Generate UUID, used for unique identification
let uuid = util.generateRandomUUID(true);
console.info("UUID: " + uuid);  // Output similar to: 84bdf796-66cc-4655-9b89-d6218d100f9c

// Binary UUID, used in special scenarios
let binaryUuid = util.generateRandomBinaryUUID(true);
console.info(JSON.stringify(binaryUuid));  // Output binary array

// UUID parsing, occasionally used
let parsedUuid = util.parseUUID("84bdf796-66cc-4655-9b89-d6218d100f9c");
console.info("Parsed UUID: " + parsedUuid);
```

### 6. Base64 Encoding/Decoding
```typescript
// Synchronous encoding, for simple scenarios
let base64 = new util.Base64();
let array = new Uint8Array([115, 49, 51]);
let encoded = base64.encodeSync(array);
console.info("Encoded result: " + encoded);  // Output: 99,122,69,122

// Synchronous encoding to string, most commonly used
let encodedStr = base64.encodeToStringSync(array);
console.info("Encoded string: " + encodedStr);  // Output: czEz

// Synchronous decoding, used with above
let decoded = base64.decodeSync(encodedStr);
console.info("Decoded result: " + decoded);  // Output: 115,49,51

// Asynchronous encoding, for large data
base64.encode(array).then((val) => {
  console.info("Async encoded result: " + val.toString());
});

// Asynchronous encoding to string, commonly used in image processing
base64.encodeToString(array).then((val) => {
  console.info("Async encoded string: " + val);
});

// Asynchronous decoding, used with above
base64.decode(encoded).then((val) => {
  console.info("Async decoded result: " + val.toString());
});

// Image to Base64, commonly used when uploading images
async function imageToBase64(imageData: Uint8Array): Promise<string> {
  try {
    return await base64.encodeToString(imageData);
  } catch (err) {
    console.error('Image to Base64 conversion failed:', err);
    return '';
  }
}

// Base64 to Image, used when downloading images
async function base64ToImage(base64Str: string): Promise<Uint8Array> {
  try {
    return await base64.decode(base64Str);
  } catch (err) {
    console.error('Base64 to Image conversion failed:', err);
    return new Uint8Array();
  }
}
```

### 7. Practical Application
Here's a simple example combining all these features:

```typescript
@Entry
@Component
struct UtilExample {
  @State message: string = '';
  @State uuid: string = '';
  @State base64Result: string = '';

  aboutToAppear() {
    // String processing
    let name = 'Zhang San';
    this.message = util.format('Welcome %s to HarmonyOS application', name);

    // Generate UUID
    this.uuid = util.generateRandomUUID(true);

    // Error handling
    try {
      let errnum = -1;
      let errorMsg = util.errnoToString(errnum);
      console.info('Error message:', errorMsg);
    } catch (err) {
      console.error('Error occurred:', err);
    }

    // Base64 test
    let testData = new Uint8Array([72, 101, 108, 108, 111]); // "Hello"
    let base64 = new util.Base64();
    this.base64Result = base64.encodeToStringSync(testData);
    console.info('Base64 encoded result:', this.base64Result);
  }

  build() {
    Column() {
      Text(this.message)
        .fontSize(20)
        .margin({ top: 20 })
      
      Text('UUID: ' + this.uuid)
        .fontSize(16)
        .margin({ top: 10 })
      
      Text('Base64: ' + this.base64Result)
        .fontSize(16)
        .margin({ top: 10 })
      
      Button('Test Async')
        .onClick(async () => {
          async function testAsync() {
            return 'Test successful';
          }
          
          const asyncCall = util.promisify(util.callbackWrapper(testAsync));
          try {
            let result = await asyncCall();
            console.info('Async test result:', result);
          } catch (err) {
            console.error('Async test error:', err);
          }
        })
      
      Button('Test Base64')
        .onClick(async () => {
          let base64 = new util.Base64();
          let testData = new Uint8Array([72, 101, 108, 108, 111]);
          
          try {
            let encoded = await base64.encodeToString(testData);
            console.info('Async encoded result:', encoded);
            
            let decoded = await base64.decode(encoded);
            console.info('Async decoded result:', decoded);
          } catch (err) {
            console.error('Base64 operation failed:', err);
          }
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

## Common Pitfalls
1. Using %s and %d incorrectly in string formatting, resulting in wrong output
2. Writing callback function parameters in wrong order when wrapping async functions, took hours to debug
3. Poor UUID generation performance due to improper cache settings
4. Error codes not matching actual error messages, spent hours checking documentation
5. Wrong input data type in Base64 encoding/decoding, causing errors
6. Program crashing due to unhandled exceptions in Base64 async operations 