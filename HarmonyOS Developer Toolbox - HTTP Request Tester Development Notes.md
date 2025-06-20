# HarmonyOS Developer Toolbox - HTTP Request Tester Development Notes

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) is a significant milestone in Huawei's distributed operating system. Its microkernel architecture enhances security and enables seamless cross-device collaboration. The developer experience is greatly improved with the new ArkTS language, enhanced UI components, and better debugging tools. Distributed capabilities allow for smooth experiences across smartphones, tablets, TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer from China, with years of experience in the HarmonyOS ecosystem. I focus on cross-platform development and performance optimization, and I enjoy sharing my technical insights and practical experiences with the community.

## Basic Knowledge

Before diving into the HTTP Request Tester, let's briefly review some basic concepts:

- **HTTP (HyperText Transfer Protocol):** The foundation of data communication for the web, supporting various request methods (GET, POST, PUT, DELETE, etc.).
- **Query Parameters:** Key-value pairs appended to the URL to pass data to the server.
- **Request Headers:** Metadata sent with HTTP requests to provide information about the request or the client.
- **Request Body:** The data sent with POST/PUT requests, often in JSON or form-data format.
- **Response Formatting:** Making server responses readable and easy to analyze.

## Preface
Recently, while working on the HarmonyOS Toolbox, I decided to add an HTTP request testing feature. The goal was to make interface testing simple and intuitive. At first, I thought it would be easy, but I soon encountered challenges with request methods, parameters, headers, bodies, and error handling.

During development, I faced many pitfalls, such as request encapsulation, error handling, and parameter processing. After several iterations, the tool became stable and user-friendly.

## I. Feature Overview

### 1.1 Main Features
- Support for GET, POST, PUT, DELETE request methods
- URL auto-completion and validation
- Support for query parameters
- Custom request headers
- Multiple request body formats (Form-data/JSON)
- Automatic JSON response formatting
- Copy response content
- Favorites feature

### 1.2 UI Features
- URL input box
- Request method selector
- Parameter configuration area
- Request body editor
- Response display area
- Send button

## II. Implementation Process

### 2.1 Development Stories

At first, I used the system's built-in http module, but quickly ran into problems. For example, if a frontend developer sent a malformed request, the program would crash. Users wanted friendly error messages, so I had to add try-catch blocks and provide detailed feedback.

A backend developer complained that the request headers were not flexible enough. I realized I needed to allow users to add custom headers, so I implemented this feature.

Handling large request bodies was another challenge. Initially, synchronous processing caused the app to freeze with big inputs. I switched to asynchronous processing and added a loading state to improve performance.

The copy-to-clipboard feature also needed refinement. Users wanted feedback when copying succeeded or failed, so I added toast notifications for better UX.

URL auto-completion was also tricky. Users wanted the tool to automatically complete URLs, so I implemented an auto-completion algorithm after several iterations.

Finally, users requested response formatting for better readability. I added automatic formatting for JSON responses to enhance the experience.

### 2.2 Request Encapsulation

1. **Import Modules**
```typescript
import http from '@ohos.net.http'
import {
  ParamItem,
  SelectOption,
  HttpHeader as CustomHttpHeader,
  HttpOptions
} from '../../common/model/http'
import { HttpUtils } from '../../utils/http'
import router from '@ohos.router'
import promptAction from '@ohos.promptAction'
import clipboard from '@ohos.pasteboard'
import { StorageUtil } from '../../utils/storage'
```

2. **Request Method Enum**
```typescript
enum HttpMethod {
  GET = 'GET',
  POST = 'POST',
  PUT = 'PUT',
  DELETE = 'DELETE'
}
```

3. **Request Options Class**
```typescript
class HttpOptions {
  method: string
  header: Record<string, string>
  extraData?: string

  constructor(method: string, header: Record<string, string>) {
    this.method = method
    this.header = header
  }
}
```

4. **Send Request Function**
```typescript
async sendRequest(): Promise<void> {
  if (this.loading) return
  this.loading = true

  let httpRequest: http.HttpRequest | null = null

  try {
    // 1. Build request URL
    let url = this.url.trim()
    if (!url.startsWith('http://') && !url.startsWith('https://')) {
      url = 'https://' + url
    }

    // 2. Add query parameters
    const enabledParams = this.queryParams.filter(p => p.enabled && p.key.trim())
    if (enabledParams.length > 0) {
      const queryString = enabledParams
        .map(p => `${encodeURIComponent(p.key.trim())}=${encodeURIComponent(p.value.trim())}`)
        .join('&')
      url += url.includes('?') ? '&' : '?'
      url += queryString
    }

    // 3. Build request headers
    const headers: Record<string, string> = {}
    this.headers
      .filter(header => header.enabled && header.key.trim())
      .forEach(header => {
        headers[header.key.trim()] = header.value.trim()
      })

    // 4. Create request options
    const options = new HttpOptions(this.method, headers)

    // 5. Handle request body
    if (this.method !== 'GET') {
      if (this.bodyType === 'form-data') {
        const formData: Record<string, string> = {}
        this.bodyParams
          .filter(param => param.enabled && param.key.trim())
          .forEach(param => {
            formData[param.key.trim()] = param.value.trim()
          })
        options.extraData = JSON.stringify(formData)
      } else {
        try {
          JSON.parse(this.jsonBody) // Validate JSON format
          options.extraData = this.jsonBody
        } catch {
          this.response = 'Request body JSON format error'
          this.loading = false
          return
        }
      }
    }

    // 6. Send request
    console.info('Request URL:', url)
    console.info('Request Options:', JSON.stringify(options))

    httpRequest = http.createHttp()
    const result = await httpRequest.request(url, options)

    // 7. Handle response
    if (result.responseCode === 200) {
      this.response = this.formatResponse(result.result as ResponseType)
    } else {
      this.response = `Request failed: ${result.responseCode}\n${JSON.stringify(result.result, null, 2)}`
    }
  } catch (error) {
    console.error('Request error:', error)
    this.response = `Request error: ${error instanceof Error ? error.message : String(error)}`
  } finally {
    if (httpRequest) {
      httpRequest.destroy()
    }
    this.loading = false
  }
}
```

### 2.3 Temporary Solutions

1. **Request Method Issue**
   - Temporary: Use http module directly
   - Problem: Crashes on format errors, poor UX
   - Final: Add try-catch and friendly error messages

2. **Request Header Issue**
   - Temporary: Fixed headers
   - Problem: Not flexible
   - Final: Add custom header feature

3. **Large Request Body Issue**
   - Temporary: Synchronous processing
   - Problem: Freezes on large bodies
   - Final: Asynchronous processing with loading state

4. **Copy Function Issue**
   - Temporary: Direct copy
   - Problem: No feedback
   - Final: Add copy success notification

5. **URL Auto-completion Issue**
   - Temporary: Manual input
   - Problem: Poor UX
   - Final: Add auto-completion feature

6. **Response Formatting Issue**
   - Temporary: Direct display
   - Problem: Not intuitive
   - Final: Add formatting feature

## III. Pitfalls and Lessons

### 3.1 Problems Encountered

1. **Request Method Issue**
   - Problem: Crashes on format errors
   - Solution: Add try-catch
   - Suggestion: Provide friendly error messages

2. **Request Header Issue**
   - Problem: Not flexible
   - Solution: Add custom headers
   - Suggestion: Let users add their own

3. **Large Request Body Issue**
   - Problem: Freezes
   - Solution: Asynchronous processing
   - Suggestion: Add loading state

4. **Copy Function Issue**
   - Problem: No feedback
   - Solution: Add notification
   - Suggestion: Handle errors

5. **URL Auto-completion Issue**
   - Problem: Poor UX
   - Solution: Add auto-completion
   - Suggestion: Optimize algorithm

6. **Response Formatting Issue**
   - Problem: Not intuitive
   - Solution: Add formatting
   - Suggestion: Support more formats

### 3.2 Optimization Suggestions

1. **Feature Optimization**
   - Support more request methods
   - Add history records
   - Support batch requests
   - Add request categorization, import/export, management, sharing, backup, etc.

2. **Performance Optimization**
   - Optimize request speed
   - Reduce memory usage
   - Release resources promptly
   - Try multithreading, algorithm optimization, result caching, asynchronous processing, etc.

3. **User Experience**
   - Add usage instructions
   - Support keyboard shortcuts
   - Add animation effects, themes, sharing, favorites, import, backup, etc.

## IV. Summary

The HTTP Request Tester tool now covers all basic functions:

- Support for multiple request methods
- Support for various parameter formats
- Real-time result preview
- One-click copy of results
- Favorites for common settings

Some minor issues remain, but it works well for most scenarios. Further optimizations will be made over time.

## V. Reference Resources
- [HarmonyOS Application Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## Welcome to Experience

This HTTP Request Tester tool is integrated into the HarmonyOS Developer Toolbox. Welcome to download and try it!

[HarmonyOS Developer Toolbox](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> Author: In the World of Development
> Email: 1743914721@qq.com
> Copyright Notice: This article is original. Please indicate the source when reprinting.

---

If you encounter similar problems, feel free to leave a comment and discuss. If you can't solve it, let's have a headache together! 