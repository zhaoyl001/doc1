# HarmonyOS Popup Usage Guide

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) represents a revolutionary step in the evolution of Huawei's distributed operating system. As someone who has been following its development closely, I can attest to the remarkable improvements in this version. The system's microkernel architecture not only enhances security but also provides unprecedented flexibility in cross-device collaboration. What excites me most about HarmonyOS 5 is its focus on developer experience—the new ArkTS language, enhanced UI components, and improved debugging tools have made development much more efficient. The distributed capabilities allow us to create truly seamless experiences across different devices, from smartphones to tablets, smart TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer based in China, with over three years of experience in the ecosystem. My journey with HarmonyOS began with its early versions, and I've witnessed its remarkable evolution. As a developer who has contributed to several open-source HarmonyOS projects, I've gained valuable insights into its architecture and best practices. I'm particularly passionate about UI/UX and component encapsulation, and I enjoy sharing my experiences through blogs and open-source contributions to help more developers embrace the HarmonyOS ecosystem.

## Basic Knowledge

Let's first understand the basics of popups in HarmonyOS:

1. **What is a Popup**
   - A popup is a temporary window that appears on the app interface to display important information, collect user input, or perform confirmation operations.
   - HarmonyOS provides various popup components to suit different needs.

2. **Main Types**
   - Standard popup: For simple messages, such as tips or warnings
   - Custom popup: Fully customizable content and style
   - Web popup: For displaying web content, such as surveys
   - Bottom popup: Slides up from the bottom, suitable for lists
   - Loading popup: Shows loading status, improves user experience

## Preface
Recently, while developing HarmonyOS applications, I often needed to use various popups. The SDK documentation looks fine, but in practice, there are many pitfalls. Here I document the issues encountered and implementation details, so that I and my colleagues can avoid these pitfalls in the future.

## I. Development Background

Our project required a survey. Initially, I wanted to write a form myself, but it was too much trouble. Fortunately, there was an existing survey system, so I decided to use a web popup to integrate it. This not only reused the existing system but also maintained a consistent user experience—a win-win.

## II. Implementation Steps

### 2.1 Basic Configuration
```typescript
// Import necessary modules, otherwise compilation will fail
import { webview } from '@kit.ArkWeb';
import { BreakpointConstants } from '../../utils/BreakpointConstants';

// Configure web cookies, very important for survey submission
webview.once("webInited", () => {
  console.log("Configuring Cookie Sync");
  webview.WebCookieManager.configCookieSync("your-survey-url", "1");
});
```

### 2.2 Custom Popup Implementation
```typescript
// Custom popup component using @CustomDialog decorator
@CustomDialog
export struct SurveyPopup {
  // Popup controller for show/hide
  controller: CustomDialogController;
  // Webview controller for loading/refreshing web content
  webController: webview.WebviewController = new webview.WebviewController();
  // Current breakpoint for responsive design
  @StorageLink('currentBreakpoint') currentBreakpoint: string = BreakpointConstants.BREAKPOINT_LG;

  build() {
    Column() {
      // Title bar, customizable style
      Row() {
        Text('Survey')
          .fontSize(20)
          .fontWeight(FontWeight.Medium)
          .fontColor($r('app.color.text_primary'))

        Blank()

        // Close button with click event
        Button({ type: ButtonType.Circle }) {
          Text('✕')
            .fontSize(20)
            .fontColor($r('app.color.text_secondary'))
        }
        .width(32)
        .height(32)
        .backgroundColor('transparent')
        .onClick(() => this.controller.close())
      }
      .width('100%')
      .padding({ left: 24, right: 16 })
      .margin({ top: 16, bottom: 16 })

      // Web content, can load any web page
      Column() {
        Web({ 
          src: 'your-survey-url', 
          controller: this.webController 
        })
        .height("90%")
      }
    }
    // Adjust popup size based on screen
    .width(this.currentBreakpoint === BreakpointConstants.BREAKPOINT_LG ? '50%' : '100%')
    .height('90%')
    .backgroundColor($r('app.color.background'))
    .borderRadius({ topLeft: 24, topRight: 24 })
  }
}
```

### 2.3 Usage Example
```typescript
// Using the popup in a page
@Entry
@Component
struct MainPage {
  // Popup controller with alignment and offset
  surveyPopupController: CustomDialogController = new CustomDialogController({
    builder: SurveyPopup(),
    alignment: DialogAlignment.Center,
    offset: { dx: 0, dy: 0 },
    customStyle: true
  });

  build() {
    Column() {
      Button('Open Survey')
        .onClick(() => {
          this.surveyPopupController.open();
        })
    }
  }
}
```

## III. Pitfalls and Issues

1. **Popup Style Issues**
   - Background color settings may not work; set the parent container's background color
   - Border radius must be set for all four corners for a proper look
   - Shadow effects need to be implemented manually; SDK does not provide them

2. **Web Loading Issues**
   - Slow web loading; add a loading indicator to avoid user confusion
   - Cookie sync may be delayed; wait for webview initialization
   - Web content height may not auto-fit; sometimes content is cut off

3. **Performance Issues**
   - Popup open/close animations may lag; optimize them
   - Web content loading affects the main thread; handle asynchronously
   - High memory usage; release resources promptly

## IV. Notes

1. **Development Configuration**
   - Configure web domain whitelist, or loading will fail
   - Handle web loading failures with user-friendly messages
   - Adapt to different screen sizes, not just phones

2. **Security Considerations**
   - Web content must use HTTPS, not HTTP
   - Prevent web injection attacks; security first
   - Protect user privacy; do not leak data

3. **Performance Optimization**
   - Use lazy loading for popup components
   - Cache web content to reduce repeated requests
   - Release resources promptly to avoid memory leaks

4. **User Experience**
   - Add loading indicators
   - Optimize close animations
   - Handle network exceptions gracefully

## V. Best Practices

1. **Popup Design**
   - Keep it simple; too much content annoys users
   - Provide a clear close button
   - Support dark mode as well as light mode

2. **Code Organization**
   - Separate popup components into individual files
   - Use TypeScript types, avoid `any`
   - Add necessary comments for maintainability

3. **Testing Suggestions**
   - Test on different screen sizes
   - Test network exceptions
   - Test memory usage to avoid leaks

## VI. Common Issues

1. **Popup Not Displaying**
   - Check if the controller is initialized
   - Ensure the popup component is registered
   - Verify style settings

2. **Web Loading Failure**
   - Check network connection
   - Check domain whitelist configuration
   - Check cookie settings

3. **Performance Issues**
   - Use lazy loading
   - Optimize web content
   - Release resources promptly

---

If you have questions, feel free to comment. This solution fits most scenarios, but for special needs, test thoroughly.

## Reference Resources
- [HarmonyOS Application Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## Welcome to Experience

This popup component has been integrated into the HarmonyOS Developer Toolbox. Welcome to download and experience it!

[HarmonyOS Developer Toolbox](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---
> Author: In the World of Development
> Copyright Notice: This article is an original work by the CSDN blogger, please include the original source link and this statement when reprinting. 