# AES Encryption/Decryption Development Notes in HarmonyOS

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) represents a revolutionary step in the evolution of Huawei's distributed operating system. As someone who has been following its development closely, I can attest to the remarkable improvements in this version. The system's microkernel architecture not only enhances security but also provides unprecedented flexibility in cross-device collaboration. What excites me most about HarmonyOS 5 is its focus on developer experience - the new ArkTS language, enhanced UI components, and improved debugging tools have made development much more efficient. The distributed capabilities allow us to create truly seamless experiences across different devices, from smartphones to tablets, smart TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer based in China, with over three years of experience in the ecosystem. My journey with HarmonyOS began with its early versions, and I've witnessed its remarkable evolution. As a developer who has contributed to several open-source HarmonyOS projects, I've gained valuable insights into its architecture and best practices. I'm particularly passionate about security tools and cryptographic functions, which led me to develop this comprehensive AES encryption/decryption module. Through my blog and GitHub repositories, I've been sharing my experiences and helping other developers navigate the exciting world of HarmonyOS development.

## Basic Knowledge

Let's first understand AES encryption:

1. What is AES
   - Advanced Encryption Standard
   - Symmetric block cipher
   - 128/192/256-bit keys
   - 128-bit block size

2. AES Modes
   - ECB (Electronic Codebook)
   - CBC (Cipher Block Chaining)
   - CFB (Cipher Feedback)
   - OFB (Output Feedback)
   - CTR (Counter)

3. Key Management
   - Secure key generation
   - Key storage
   - Key rotation
   - Key backup

4. Security Considerations
   - Use strong keys
   - Choose proper mode
   - Handle IV properly
   - Protect keys

5. Best Practices
   - Use CBC or CTR mode
   - Generate random IV
   - Use proper padding
   - Validate input

## Preface
Recently, while developing the HarmonyOS Developer Toolbox, we wanted to add an AES encryption/decryption feature. This feature is mainly used to encrypt and decrypt data using the AES algorithm. We chose to use the crypto-js library for its reliability and ease of use.

## I. Feature Description

### 1.1 Main Features
- AES encryption
- AES decryption
- Multiple modes
- Key generation
- IV management
- File support
- Batch processing

### 1.2 Interface Features
- Text input field
- File selection
- Mode selection
- Key input
- IV input
- Result display
- History view

## II. Implementation Process

### 2.1 Implementation Principle

AES encryption/decryption involves several steps:

1. Key Generation
   - Generate random key
   - Derive from password
   - Import existing key
   - Validate key

2. IV Management
   - Generate random IV
   - Store IV with data
   - Validate IV
   - Handle IV properly

3. Encryption Process
   - Choose mode
   - Apply padding
   - Encrypt data
   - Format output

4. Decryption Process
   - Extract IV
   - Choose mode
   - Decrypt data
   - Remove padding

5. File Handling
   - Read file content
   - Process in chunks
   - Handle errors
   - Show progress

### 2.2 Code Implementation

```typescript
import CryptoJS from 'crypto-js';

@Entry
@Component
struct AESEncryptor {
  @State input: string = '';               // Input text
  @State output: string = '';              // Output text
  @State key: string = '';                 // Encryption key
  @State iv: string = '';                  // Initialization vector
  @State mode: string = 'CBC';             // Encryption mode
  @State isProcessing: boolean = false;    // Processing status
  @State history: string[] = [];           // Operation history

  // Generate random key
  private generateKey() {
    const key = CryptoJS.lib.WordArray.random(32);
    this.key = key.toString();
  }

  // Generate random IV
  private generateIV() {
    const iv = CryptoJS.lib.WordArray.random(16);
    this.iv = iv.toString();
  }

  // Encrypt data
  private encrypt() {
    try {
      this.isProcessing = true;

      // Validate input
      if (!this.input || !this.key) {
        throw new Error('Input and key are required');
      }

      // Prepare key and IV
      const key = CryptoJS.enc.Utf8.parse(this.key);
      const iv = CryptoJS.enc.Utf8.parse(this.iv || '');

      // Encrypt
      const encrypted = CryptoJS.AES.encrypt(this.input, key, {
        iv: iv,
        mode: CryptoJS.mode[this.mode],
        padding: CryptoJS.pad.Pkcs7
      });

      // Format output
      this.output = encrypted.toString();
      
      // Update history
      this.history.unshift({
        type: 'encrypt',
        input: this.input,
        output: this.output,
        timestamp: new Date().toISOString()
      });

      // Limit history size
      if (this.history.length > 10) {
        this.history.pop();
      }

    } catch (err) {
      console.error('Encryption failed:', err);
      throw err;
    } finally {
      this.isProcessing = false;
    }
  }

  // Decrypt data
  private decrypt() {
    try {
      this.isProcessing = true;

      // Validate input
      if (!this.input || !this.key) {
        throw new Error('Input and key are required');
      }

      // Prepare key and IV
      const key = CryptoJS.enc.Utf8.parse(this.key);
      const iv = CryptoJS.enc.Utf8.parse(this.iv || '');

      // Decrypt
      const decrypted = CryptoJS.AES.decrypt(this.input, key, {
        iv: iv,
        mode: CryptoJS.mode[this.mode],
        padding: CryptoJS.pad.Pkcs7
      });

      // Format output
      this.output = decrypted.toString(CryptoJS.enc.Utf8);
      
      // Update history
      this.history.unshift({
        type: 'decrypt',
        input: this.input,
        output: this.output,
        timestamp: new Date().toISOString()
      });

      // Limit history size
      if (this.history.length > 10) {
        this.history.pop();
      }

    } catch (err) {
      console.error('Decryption failed:', err);
      throw err;
    } finally {
      this.isProcessing = false;
    }
  }
}
```

## III. Development Experience

### 3.1 Problems Encountered

1. Key Management
   - Problem: Secure key storage
   - Solution: Used secure storage

2. Mode Selection
   - Problem: Mode compatibility
   - Solution: Added mode validation

3. Performance Issues
   - Problem: Large file handling
   - Solution: Implemented chunking

4. User Experience
   - Problem: Error handling
   - Solution: Added clear messages

### 3.2 Optimization Suggestions

1. Feature Optimization
   - Add more modes
   - Support key derivation
   - Add file encryption
   - Support batch processing

2. Performance Optimization
   - Optimize file handling
   - Improve chunking
   - Cache results
   - Reduce memory usage

3. User Experience
   - Add drag and drop
   - Support batch files
   - Add progress bar
   - Improve feedback

4. Security
   - Enhance key management
   - Add input validation
   - Implement rate limiting
   - Improve logging

## IV. Summary

This AES tool has all the basic features and can:

- Encrypt and decrypt data
- Handle multiple modes
- Manage keys and IVs
- Process files

## V. Reference Resources

- [HarmonyOS Application Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)
- [crypto-js Documentation](https://github.com/brix/crypto-js)

## Welcome to Experience

This AES tool has been integrated into the HarmonyOS Developer Toolbox. Welcome to download and experience it!

[HarmonyOS Developer Toolbox](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> Author: In the World of Development
> Email: 1743914721@qq.com
> Copyright Notice: This article is an original work by the CSDN blogger, please include the original source link and this statement when reprinting. 