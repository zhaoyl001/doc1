# HarmonyOS Privacy Popup Development Practice

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) represents a revolutionary step in the evolution of Huawei's distributed operating system. As someone who has been following its development closely, I can attest to the remarkable improvements in this version. The system's microkernel architecture not only enhances security but also provides unprecedented flexibility in cross-device collaboration. What excites me most about HarmonyOS 5 is its focus on developer experience—the new ArkTS language, enhanced UI components, and improved debugging tools have made development much more efficient. The distributed capabilities allow us to create truly seamless experiences across different devices, from smartphones to tablets, smart TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer based in China, with over three years of experience in the ecosystem. My journey with HarmonyOS began with its early versions, and I've witnessed its remarkable evolution. As a developer who has contributed to several open-source HarmonyOS projects, I've gained valuable insights into its architecture and best practices. I'm particularly passionate about privacy protection, UI/UX, and compliance, and I enjoy sharing my experiences through blogs and open-source contributions to help more developers embrace the HarmonyOS ecosystem.

## Preface
Recently, while developing HarmonyOS applications, I found that privacy popups are an unavoidable feature. They are not only crucial for user experience but also for app compliance. After some exploration and practice, I finally implemented a privacy popup that I am quite satisfied with. Here I share my development insights.

## I. Development Background

When developing HarmonyOS apps, privacy popups are extremely important. They impact both user experience and compliance. This article details how to implement a beautiful and practical custom privacy popup in HarmonyOS apps.

## II. Implementation Steps

### 2.1 Creating the Privacy Popup Component

First, create a custom privacy popup component `PrivacyDialog.ets`:

```typescript
@CustomDialog
export struct PrivacyDialog {
  controller: CustomDialogController;
  context: common.UIAbilityContext = getContext(this) as common.UIAbilityContext;
  confirm: () => void = () => {};
  cancel: () => void = () => {};

  build() {
    Column() {
      // Title
      Text('Privacy Policy')
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
        .margin({ top: 16, bottom: 16 })
        .textAlign(TextAlign.Center)
        .fontColor($r('app.color.text_primary'))

      // Content
      Text() {
        Span('This app values and protects all users\' personal privacy. To provide you with more accurate and personalized services, this app will use and disclose your personal information in accordance with the privacy policy')
          .fontSize(16)
          .fontColor($r('app.color.text_secondary'))

        Span(' as stipulated. You can read the ')
          .fontSize(16)
          .fontColor($r('app.color.text_secondary'))
          
        Span('Service Agreement')
          .fontSize(16)
          .fontColor($r('app.color.brand_color'))
          .onClick(() => {
            this.controller.close();
            router.pushUrl({
              url: 'pages/UserAgreementDetail',
              params: { needShowDialog: true }
            });
          })
          
        Span(' and ')
          .fontSize(16)
          .fontColor($r('app.color.text_secondary'))
          
        Span('Privacy Policy')
          .fontSize(16)
          .fontColor($r('app.color.brand_color'))
          .onClick(() => {
            this.controller.close();
            router.pushUrl({
              url: 'pages/PrivacyPolicyDetail',
              params: { needShowDialog: true }
            });
          })
      }
      .margin({ left: 16, right: 16 })
      .textAlign(TextAlign.Center)

      // Buttons
      Row() {
        Button('Agree')
          .width('45%')
          .height(40)
          .backgroundColor($r('app.color.brand_color'))
          .fontColor(Color.White)
          .onClick(() => {
            this.confirm();
            this.controller.close();
          })

        Button('Disagree and Exit')
          .width('45%')
          .height(40)
          .fontColor($r('app.color.text_secondary'))
          .backgroundColor($r('app.color.card_background'))
          .onClick(() => {
            this.cancel();
            this.controller.close();
          })
      }
      .width('100%')
      .justifyContent(FlexAlign.SpaceAround)
      .margin({ top: 40, bottom: 16 })
    }
    .width('90%')
    .backgroundColor($r('app.color.card_background'))
    .borderRadius(16)
    .padding(16)
  }
}
```

### 2.2 Using in the Welcome Page

Integrate the privacy popup in the welcome page `Welcome.ets`:

```typescript
@Entry
@Component
struct Welcome {
  @State showPrivacyDialog: boolean = false;
  private context = getContext(this) as common.UIAbilityContext;
  private preferences: dataPreferences.Preferences | null = null;
  
  private dialogController: CustomDialogController = new CustomDialogController({
    builder: PrivacyDialog({
      context: this.context,
      confirm: async () => {
        try {
          if (this.preferences) {
            await this.preferences.put('privacy_agreed', true);
            await this.preferences.flush();
          }
          router.replaceUrl({ url: 'pages/Index' });
        } catch (error) {
          console.error('Update privacy agreement flag failed:', error);
        }
      },
      cancel: () => {
        this.context.terminateSelfWithResult({
          resultCode: 0,
          want: {}
        });
      }
    }),
    alignment: DialogAlignment.Center,
    offset: { dx: 0, dy: 0 },
    customStyle: true,
    autoCancel: false
  });

  async aboutToAppear() {
    try {
      this.preferences = await dataPreferences.getPreferences(this.context, 'app_settings');
      const hasAgreed = await this.preferences.get('privacy_agreed', false);
      
      if (!hasAgreed) {
        this.showPrivacyDialog = true;
        this.dialogController.open();
      } else {
        setTimeout(() => {
          router.replaceUrl({ url: 'pages/Index' });
        }, 1100);
      }
    } catch (error) {
      console.error('Initialize preferences failed:', error);
    }
  }
}
```

## III. Pitfalls and Issues

### 3.1 UI Design
- The initial popup layout was random; after referencing mainstream apps, it became more beautiful
- Privacy policy text was too long at first; users wouldn't read it, so it was simplified
- Button styles were not prominent enough at first; took several revisions to be satisfied
- Dark mode adaptation took a lot of time

### 3.2 Interaction Experience
- When viewing detailed privacy policy, the popup was not closed at first, causing user confusion
- The navigation logic for Service Agreement and Privacy Policy was revised several times
- The timing for redirecting after agreement was tricky; added a delay later
- No confirmation for exit at first when disagreeing

### 3.3 Data Storage
- Exception handling for Preferences storage was poor at first, causing data loss
- The issue of repeated popups took a long time to fix
- The reset privacy setting feature was added later due to user demand

## IV. Notes

1. **Permission Management**
   - Storage permissions must be requested, or data cannot be saved
   - The permission request process should be user-friendly

2. **User Experience**
   - Popup content should be simple and direct
   - Operation guidance should be clear
   - Detailed policies should be easy to find but not too prominent

3. **Compliance**
   - Follow laws and regulations
   - Protect user privacy—this is the bottom line
   - Clearly state necessary information

## V. Summary

This privacy popup feature went through several iterations and many pitfalls. But with persistence, the final result is satisfactory. If you are working on a similar feature, I hope this article helps you.

## VI. Reference Resources

- [HarmonyOS Application Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## Welcome to Experience

This privacy popup feature has been integrated into the HarmonyOS Developer Toolbox. Welcome to download and experience it!

[HarmonyOS Developer Toolbox](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---
> Author: In the World of Development
> Email: 1743914721@qq.com
> Copyright Notice: This article is an original work by the CSDN blogger, please include the original source link and this statement when reprinting. 