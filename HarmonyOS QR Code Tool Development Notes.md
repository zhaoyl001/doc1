# HarmonyOS QR Code Tool Development Notes

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) represents a revolutionary step in the evolution of Huawei's distributed operating system. As someone who has been following its development closely, I can attest to the remarkable improvements in this version. The system's microkernel architecture not only enhances security but also provides unprecedented flexibility in cross-device collaboration. What excites me most about HarmonyOS 5 is its focus on developer experience—the new ArkTS language, enhanced UI components, and improved debugging tools have made development much more efficient. The distributed capabilities allow us to create truly seamless experiences across different devices, from smartphones to tablets, smart TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer based in China, with over three years of experience in the ecosystem. My journey with HarmonyOS began with its early versions, and I've witnessed its remarkable evolution. As a developer who has contributed to several open-source HarmonyOS projects, I've gained valuable insights into its architecture and best practices. I'm particularly passionate about tool development, QR code technology, and user experience, and I enjoy sharing my experiences through blogs and open-source contributions to help more developers embrace the HarmonyOS ecosystem.

## Preface
Recently, while working on the HarmonyOS Toolbox, I was inspired to add a QR code tool. Honestly, I was quite confident at first—"It's just a simple QR code, how hard can it be?" But once I started, I realized I was too naive! Various formats, parameters, saving, scanning—it was overwhelming. The scanning feature especially kept crashing, making me want to smash my keyboard.

I encountered many pitfalls while developing this tool, such as scan encapsulation, image saving, permission handling, etc. I remember one night debugging the save feature until 3 a.m., almost smashing my computer. But hard work paid off, and now it works smoothly. Seeing users say "It's really useful" makes me very happy!

## I. Feature Description

### 1.1 Main Features
- Support QR code generation
- Support QR code scanning
- Support selecting images from the gallery
- Support saving QR codes to the gallery
- Support copying scan results
- Favorites feature

### 1.2 UI Features
- Generate/Scan tab switch
- Content input box
- QR code preview
- Scan button
- Save button
- Copy button

## II. Implementation Process

### 2.1 Development Stories

#### Story 1: The Scanning Crash Saga
At first, I used the system's built-in scanning module, thinking, "Isn't this just calling an API? Easy!" But all sorts of problems followed. The funniest was when a user scanned a blurry QR code and the app crashed. The user was furious: "What is this? I want a friendly error message!" I thought, "Isn't that just an error message? Easy!" But once I started, I realized I needed to add try-catch and provide specific error info. After several revisions and adding a loading animation, things finally improved.

#### Story 2: The Awkward Save Moment
Another time, a user said the save feature didn't work well. At first, I felt wronged: "Isn't this simple?" But after trying it myself, I realized there was indeed a problem. The most awkward part was that there was no prompt when saving, and sometimes it failed. The user said, "I tried to save for a long time, but didn't know where it went. So annoying!" I quickly revised it several times and added a success prompt. Now, thinking back, I'm grateful for that user's feedback—otherwise, I wouldn't have noticed the issue.

After some research, I found that saving is actually quite complex:
1. First, convert the QR code to an image format, which may cause memory issues (sometimes OOM, really frustrating)
2. Then, request gallery permissions, which can be tricky (sometimes the popup doesn't appear, really annoying)
3. Finally, handle various exceptions when saving to the gallery

The funniest was when I tested and saved successfully, but couldn't find the image. After a long time, I realized it was a path issue. HarmonyOS's gallery path is different from Android, and you need to use a special API. This pitfall was both funny and frustrating.

#### Story 3: The Big Image Freeze Crisis
The funniest was handling big images. At first, I didn't consider performance, and a user sent a huge image, causing the app to freeze. I thought, "It's just an image, easy!" But once I started, I realized I needed to consider memory usage and do async processing. After several revisions and adding a loading state, things finally improved. Now, thinking back, I'm grateful for that user's "big image"—otherwise, I wouldn't have noticed the performance issue.

#### Story 4: The Copy Feature Surprise
Another user said the copy feature didn't work well. At first, I felt wronged: "Isn't this simple?" But after trying it myself, I realized there was indeed a problem. The most awkward part was that there was no prompt when copying, and sometimes it failed. The user said, "I tried to copy for a long time, but didn't know if it succeeded. So annoying!" I quickly revised it several times and added a success prompt. Now, thinking back, I'm grateful for that user's feedback—otherwise, I wouldn't have noticed the issue.

### 2.2 Core Feature Implementation

1. Import modules
```typescript
import router from '@ohos.router'
import promptAction from '@ohos.promptAction'
import { scanCore, scanBarcode } from '@kit.ScanKit'
import { hilog } from '@kit.PerformanceAnalysisKit'
import pasteboard from '@ohos.pasteboard'
import common from '@ohos.app.ability.common'
import image from '@ohos.multimedia.image'
import fileio from '@ohos.fileio'
import { photoAccessHelper } from '@kit.MediaLibraryKit'
import { StorageUtil } from '../../utils/storage'
import { componentSnapshot } from '@kit.ArkUI'
import { fileIo } from '@kit.CoreFileKit'
import { BusinessError } from '@kit.BasicServicesKit'
```

2. Scan feature implementation
```typescript
private startScan() {
  try {
    let options: scanBarcode.ScanOptions = {
      scanTypes: [scanCore.ScanType.ALL],
      enableMultiMode: true,
      enableAlbum: true
    }

    scanBarcode.startScanForResult(getContext(this), options, (error, result: scanBarcode.ScanResult) => {
      if (error) {
        hilog.error(0x0001, 'QRCodeTool', `Failed to get ScanResult. Code: ${error.code}, message: ${error.message}`)
        promptAction.showToast({ message: 'Scan failed' })
        return
      }

      if (result) {
        try {
          console.info('Scan result:', JSON.stringify(result))
          
          let scanText = ''
          if (typeof result === 'object') {
            scanText = result['originalValue'] || result['result'] || result['value'] || ''
          }

          if (scanText) {
            this.scanContent = scanText
            hilog.info(0x0001, 'QRCodeTool', `Scan success: ${scanText}`)
            promptAction.showToast({ message: 'Scan successful' })
          } else {
            hilog.error(0x0001, 'QRCodeTool', 'Empty scan result')
            promptAction.showToast({ message: 'No scan content found' })
          }
        } catch (err) {
          hilog.error(0x0001, 'QRCodeTool', `Parse result error: ${err}`)
          promptAction.showToast({ message: 'Failed to parse scan result' })
        }
      }
    })
  } catch (error) {
    hilog.error(0x0001, 'QRCodeTool', `Failed to startScanForResult. Code: ${error.code}, message: ${error.message}`)
    promptAction.showToast({ message: 'Failed to start scan' })
  }
}
```

3. Save feature implementation
```typescript
// Pack PixelMap as jpg
async packingPixelMap2Jpg(pixelMap: PixelMap): Promise<ArrayBuffer> {
  const imagePackerApi = image.createImagePacker();
  const packOpts: image.PackingOption = { format: "image/jpeg", quality: 98 };
  let imageBuffer: ArrayBuffer = new ArrayBuffer(1);
  try {
    imageBuffer = await imagePackerApi.packing(pixelMap, packOpts);
  } catch (err) {
    console.error(`Invoke packingPixelMap2Jpg failed, err: ${JSON.stringify(err)}`);
  }
  return imageBuffer;
}

// Save image to gallery
async savePhotoToGallery(context: common.UIAbilityContext) {
  let helper = photoAccessHelper.getPhotoAccessHelper(context);
  try {
    const imageBuffer = await this.packingPixelMap2Jpg(this.pixmap as image.PixelMap)
    
    let uri = await helper.createAsset(photoAccessHelper.PhotoType.IMAGE, 'jpg');
    let file = await fileIo.open(uri, fileIo.OpenMode.READ_WRITE | fileIo.OpenMode.CREATE);
    await fileIo.write(file.fd, imageBuffer);
    await fileIo.close(file.fd);
    promptAction.showToast({ message: 'Saved to gallery!' });
  }
  catch (error) {
    const err: BusinessError = error as BusinessError;
    console.error(`Failed to save photo. Code is ${err.code}, message is ${err.message}`);
  }
}
```

4. Copy feature implementation
```typescript
private async copyToClipboard(content: string) {
  try {
    const pasteData = pasteboard.createData(pasteboard.MIMETYPE_TEXT_PLAIN, content)
    const systemPasteboard = pasteboard.getSystemPasteboard()
    await systemPasteboard.setData(pasteData)
    await promptAction.showToast({ message: 'Copied to clipboard' })
  } catch (error) {
    console.error('Copy failed:', error)
    await promptAction.showToast({ message: 'Copy failed' })
  }
}
```

### 2.3 Temporary Solutions

1. Scan issues
   - Temporary: Use scan module directly, simple and crude
   - Problem: Crashes on format error, poor user experience
   - Final: Add try-catch, provide friendly prompts

2. Save issues
   - Temporary: Save directly, simple
   - Problem: No prompt, poor experience, incomplete permission handling
   - Final:
     * Add save success prompt
     * Improve permission request process
     * Optimize error handling
     * Ensure files are properly closed
     * Add detailed error prompts

3. Big image issues
   - Temporary: Synchronous processing, simple
   - Problem: Big images cause freezes
   - Final: Async processing, add loading state

4. Copy issues
   - Temporary: Copy directly, simple
   - Problem: No prompt, poor experience
   - Final: Add copy success prompt

## III. Pitfalls and Issues

### 3.1 Problems Encountered

1. Scan issues
   - Problem: Crashes on format error
   - Solution: Add try-catch
   - Suggestion: Provide friendly prompts

2. Save issues
   - Problem: Save failed
   - Solution: Use SaveButton and componentSnapshot
   - Suggestion: Pay attention to permission requests

3. Big image issues
   - Problem: Causes freezes
   - Solution: Async processing
   - Suggestion: Add loading state

4. Copy issues
   - Problem: No prompt
   - Solution: Add prompt
   - Suggestion: Handle errors

### 3.2 Optimization Suggestions

1. Feature optimization
   - Support more QR code formats
   - Add history
   - Support batch scanning
   - Beautify QR codes, custom styles, batch generation, etc.

2. Performance optimization
   - Optimize scan speed
   - Reduce memory usage
   - Release resources promptly
   - Try multithreading, algorithm optimization, result caching, async processing

3. User experience
   - Add usage instructions
   - Support shortcuts
   - Add animation effects, themes, sharing, favorites, import, backup, etc.

## IV. Summary

This QR code tool covers all the basics:

- Support QR code generation
- Support QR code scanning
- Support saving to gallery
- Support copying results
- Favorites for common settings

To be honest, developing this tool was a love-hate process. I loved seeing it grow from nothing to something useful, but hated all the pitfalls along the way. Looking back, those pitfalls were worth it! Without them, I wouldn't have found so many areas to optimize.

Some edge cases are still not fully resolved, such as occasional slow scanning or UI stutter when saving big images (and sometimes strange errors pop up, which is really frustrating). But it works for most scenarios, and user feedback has been positive. I'll keep optimizing as time allows—after all, good products are polished over time.

## V. Reference Resources

- [HarmonyOS Application Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## Welcome to Experience

This QR code tool has been integrated into the HarmonyOS Developer Toolbox. Welcome to download and experience it! If you encounter any issues, feel free to give feedback, and I will continue to improve it!

[HarmonyOS Developer Toolbox](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---
> Author: In the World of Development
> Email: 1743914721@qq.com
> Copyright Notice: This article is an original work by the CSDN blogger, please include the original source link and this statement when reprinting.

---
If you encounter similar issues, feel free to comment and discuss. Let's improve and grow together! 