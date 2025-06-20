# Huawei Account Login Feature Implementation Guide

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) represents a revolutionary step in the evolution of Huawei's distributed operating system. As someone who has been following its development closely, I can attest to the remarkable improvements in this version. The system's microkernel architecture not only enhances security but also provides unprecedented flexibility in cross-device collaboration. What excites me most about HarmonyOS 5 is its focus on developer experience—the new ArkTS language, enhanced UI components, and improved debugging tools have made development much more efficient. The distributed capabilities allow us to create truly seamless experiences across different devices, from smartphones to tablets, smart TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer based in China, with over three years of experience in the ecosystem. My journey with HarmonyOS began with its early versions, and I've witnessed its remarkable evolution. As a developer who has contributed to several open-source HarmonyOS projects, I've gained valuable insights into its architecture and best practices. I'm particularly passionate about account integration and secure authentication, and I enjoy sharing my experiences through blogs and open-source contributions to help more developers embrace the HarmonyOS ecosystem.

## Basic Knowledge

Let's first understand the basics of Huawei Account Login:

1. **What is Huawei Account Login**
   - Allows users to log in to your app with their Huawei account, eliminating the need for registration and password management.
   - Similar to WeChat/QQ login, but for Huawei ecosystem.

2. **Key Concepts**
   - **UnionID**: Globally unique, identifies the same user across apps. Not the same as OpenID.
   - **OpenID**: Unique per app, changes with each app.
   - **AuthorizationCode**: Temporary credential, used by the backend to exchange for a token.
   - **AccessToken**: Used to call APIs, should not be exposed to the frontend.

## Preface
Recently, our project needed to integrate Huawei Account Login. The SDK documentation looked fine, but in practice, there were many pitfalls. Here I document the issues encountered and implementation details, so that I and my colleagues can avoid these pitfalls in the future.

## I. Development Background

Most of our users use Huawei phones. The boss said, "Add Huawei Account Login," so here is this note. SDK integration is not hard, but handling edge cases and exceptions is the real challenge.

## II. Implementation Steps

### 2.1 Basic Configuration
```typescript
// Import all necessary modules, or compilation will fail
import { LoginWithHuaweiIDButton, loginComponentManager, authentication } from '@kit.AccountKit';
import { BusinessError } from '@kit.BasicServicesKit';
import { hilog } from '@kit.PerformanceAnalysisKit';
import { util } from '@kit.ArkTS';
import { RDBUtils } from '../../utils/rdbutils';
import { UserInfo } from '../../utils/rdbutils';
import dataPreferences from '@ohos.data.preferences';
import common from '@ohos.app.ability.common';
import promptAction from '@ohos.promptAction';
import router from '@ohos.router';
```

### 2.2 Login Feature Implementation
```typescript
@Entry
@Component
struct PreviewLoginButtonPage {
  // Database utility instance
  private rdbUtils: RDBUtils = RDBUtils.getInstance(getContext(this));
  private context: common.Context = getContext(this);
  
  // State variables
  @State userInfo: UserInfo | null = null;
  @State isLoggedIn: boolean = false;
  @State showUserInfo: boolean = false;
  @State loading: boolean = false;

  // Login button controller
  controller: loginComponentManager.LoginWithHuaweiIDButtonController =
    new loginComponentManager.LoginWithHuaweiIDButtonController()
      .onClickLoginWithHuaweiIDButton(async (error: BusinessError, response: loginComponentManager.HuaweiIDCredential) => {
        if (error) {
          hilog.error(0x0000, 'testTag',
            `Login failed. Error code: ${error.code}, Message: ${error.message}`);
          return;
        }

        if (response) {
          this.loading = true;
          try {
            // Get login credentials
            const authCode = response.authorizationCode;
            const openID = response.openID;
            const unionID = response.unionID;
            const idToken = response.idToken;

            // Get user details
            const authRequest = new authentication.HuaweiIDProvider()
              .createAuthorizationWithHuaweiIDRequest();
            authRequest.scopes = ['profile'];
            authRequest.permissions = ['serviceauthcode'];
            authRequest.forceAuthorization = true;
            authRequest.state = util.generateRandomUUID();
            
            const controller = new authentication.AuthenticationController(this.context);
            const data = await controller.executeRequest(authRequest);
            
            // Handle user info
            const userInfo: UserInfo = {
              avatarUri: data.data?.avatarUri || '',
              nickName: data.data?.nickName || '',
              unionID: data.data?.unionID || '',
              openID: data.data?.openID || '',
              authorizationCode: data.data?.authorizationCode || '',
              createTime: Date.now(),
              memo: 'Huawei Account Login'
            };

            // Save user info
            await this.saveUserInfo(userInfo);
            
            // Update state
            this.isLoggedIn = true;
            this.showUserInfo = true;
          } catch (err) {
            hilog.error(0x0000, 'testTag', `Login handling failed: ${err}`);
          } finally {
            this.loading = false;
          }
        }
      });
}
```

### 2.3 User Info Management
```typescript
private async saveUserInfo(userInfo: UserInfo) {
  try {
    // Check if user already exists
    const existingUser = await this.rdbUtils.queryUserByUnionID(userInfo.unionID);

    if (existingUser) {
      // Update existing user info
      userInfo.id = existingUser.id;
      await this.rdbUtils.updateUser(userInfo);
    } else {
      // Insert new user info
      await this.rdbUtils.insertUser(userInfo);
    }

    // Save current user's unionID
    await this.saveCurrentUserUnionID(userInfo.unionID);

    // Show login success toast
    promptAction.showToast({
      message: 'Login successful',
      duration: 2000
    });

    // Delay and return to home
    setTimeout(() => {
      router.back();
    }, 2000);
  } catch (error) {
    hilog.error(0x0000, 'testTag', `Failed to save user info: ${error}`);
    promptAction.showToast({
      message: 'Login failed',
      duration: 2000
    });
  }
}
```

## III. Pitfalls and Issues

- Login state not persisted; users had to log in again after restarting the app.
- Exceptions not handled in login callback; UI sometimes froze.
- Multi-device login state not synchronized; info got messy when switching devices.
- No loading indicator on slow network; users thought the button was unresponsive.
- Incomplete error code handling; some errors were unclear to users.
- No image caching; avatar was reloaded every time, poor experience.

## IV. Usage Example

```typescript
// 1. Initialize login page
@Entry
@Component
struct LoginPage {
  build() {
    Column() {
      // Login button
      LoginWithHuaweiIDButton({
        params: {
          style: loginComponentManager.Style.BUTTON_RED,
          extraStyle: {
            buttonStyle: new loginComponentManager.ButtonStyle()
              .loadingStyle({
                show: this.loading
              })
          },
          borderRadius: 24,
          loginType: loginComponentManager.LoginType.ID,
          supportDarkMode: true,
          verifyPhoneNumber: true
        },
        controller: this.controller
      })
    }
  }
}

// 2. Check login status
private async checkLoginStatus() {
  try {
    this.loading = true;
    const currentUnionID = await this.getCurrentUserUnionID();
    if (currentUnionID) {
      const user = await this.rdbUtils.queryUserByUnionID(currentUnionID);
      if (user) {
        this.userInfo = user;
        this.isLoggedIn = true;
        this.showUserInfo = true;
      }
    }
  } catch (error) {
    hilog.error(0x0000, 'testTag', `Failed to check login status: ${error}`);
  } finally {
    this.loading = false;
  }
}

// 3. Logout
private async logout() {
  try {
    await this.clearCurrentUserUnionID();
    this.isLoggedIn = false;
    this.showUserInfo = false;
    this.userInfo = null;
    promptAction.showToast({
      message: 'Logged out',
      duration: 2000
    });
  } catch (error) {
    hilog.error(0x0000, 'testTag', `Logout failed: ${error}`);
  }
}
```

## V. Notes

- Make sure all configurations in the Huawei Developer Alliance are correct: package name, signature, permissions—any mistake will cause failure.
- Encrypt tokens and sensitive information; do not store them in plain text.
- Add loading indicators for slow networks; don't keep users waiting without feedback.
- Provide user-friendly error messages; don't just show "Login failed," give some reason.
- Ensure multi-device synchronization; don't let info get messy when users switch devices.
- Never hardcode any IDs or secrets in the code; security first.

## VI. Backend Implementation Notes

> This is only the frontend implementation; you need to handle the backend yourself. For example, token exchange, user info storage, permission checks, and API security must all be handled by the backend. Don't rely solely on the frontend—security is critical.

### 6.1 Backend Service Requirements

- **User authentication service**: Validate Huawei account credentials, manage sessions, token refresh, login state sync.
- **User info service**: Store and manage user info, permissions, sync, and updates.
- **Security service**: Data encryption, sensitive info protection, access control, prevent data leaks.

### 6.2 Backend API Endpoints

```typescript
// 1. User authentication API
POST /api/auth/login
Request: {
  authorizationCode: string;  // Huawei account authorization code
  openID: string;            // Huawei account OpenID
  unionID: string;           // Huawei account UnionID
}
Response: {
  token: string;             // User access token
  refreshToken: string;      // Refresh token
  expiresIn: number;         // Expiry time
}

// 2. User info API
GET /api/user/info
Request: {
  token: string;             // User access token
}
Response: {
  userInfo: {
    avatarUri: string;       // Avatar
    nickName: string;        // Nickname
    unionID: string;         // UnionID
    openID: string;          // OpenID
    // Other user info
  }
}

// 3. Refresh token API
POST /api/auth/refresh
Request: {
  refreshToken: string;      // Refresh token
}
Response: {
  token: string;             // New access token
  expiresIn: number;         // Expiry time
}
```

### 6.3 Security Considerations

- All APIs must use HTTPS; never use plain HTTP.
- Encrypt tokens and sensitive data; never store them in plain text.
- Prevent SQL injection, brute force attacks, and other common security issues.
- Manage login devices, token expiry, and anomaly detection.

### 6.4 Deployment Suggestions

- Always have backups for servers and databases.
- Set up logging, monitoring, and alerts to catch issues early.
- Use Redis for caching; protect APIs from being overwhelmed.
- Implement firewalls, rate limiting, and blacklists.

### 6.5 Development Suggestions

- Use mainstream frameworks, ORM, API docs, and logging systems; don't reinvent the wheel.
- Standardize error handling; don't skimp on logs.
- Implement unit, stress, and automated tests.
- Apply security patches promptly.

### 6.6 Notes

- You must implement backend services yourself; the frontend is just a "shell."
- Apply for Huawei Developer Alliance account and API permissions in advance.
- Ensure data security, service monitoring, and emergency plans.
- Important: Never hardcode secrets, tokens, or user data in the frontend!

---

If you have questions, feel free to comment. This solution fits most scenarios, but for special needs, test thoroughly.

## Reference Resources
- [HarmonyOS Application Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## Welcome to Experience

This feature has been integrated into the HarmonyOS Developer Toolbox. Welcome to download and experience it!

[HarmonyOS Developer Toolbox](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---
> Author: In the World of Development
> Email: 1743914721@qq.com
> Copyright Notice: This article is an original work by the CSDN blogger, please include the original source link and this statement when reprinting. 