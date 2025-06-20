# HarmonyOS Developer Toolbox - JSON Formatter Development Notes

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) is a major milestone in Huawei's distributed operating system. Its microkernel architecture enhances security and enables seamless cross-device collaboration. The developer experience is greatly improved with the new ArkTS language, enhanced UI components, and better debugging tools. Distributed capabilities allow for smooth experiences across smartphones, tablets, TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer from China, with years of experience in the HarmonyOS ecosystem. I focus on cross-platform development and performance optimization, and I enjoy sharing my technical insights and practical experiences with the community.

## Basic Knowledge

Before diving into the JSON Formatter, let's briefly review some basic concepts:

- **JSON (JavaScript Object Notation):** A lightweight data-interchange format, easy for humans to read and write, and easy for machines to parse and generate.
- **Indentation:** Refers to the number of spaces or tabs used to format JSON for better readability.
- **Error Handling:** Ensuring that invalid JSON or user mistakes do not crash the application, but instead provide friendly feedback.

## Preface
Recently, while working on the HarmonyOS Toolbox, I decided to add a JSON formatting feature. The goal was to make messy JSON data neat and readable. At first, I thought it would be simple, but I soon encountered challenges with indentation, line breaks, and error handling.

During development, I faced many pitfalls, such as JSON parsing, error handling, and indentation settings. After several iterations, the tool became stable and user-friendly.

## I. Feature Overview

### 1.1 Main Features
- Format JSON text
- Customizable indentation (number of spaces)
- One-click copy of results
- Error prompts for invalid JSON
- Real-time preview of formatted results
- Favorites for commonly used settings

### 1.2 UI Features
- JSON input box
- Indentation setting
- Format button
- Result display area
- Copy button

## II. Implementation Process

### 2.1 Development Stories

At first, I used the system's built-in `JSON.parse`, but quickly ran into problems. For example, if a user entered malformed JSON, the program would crash. Users wanted friendly error messages, so I had to add try-catch blocks and provide detailed feedback.

A frontend developer friend complained that the indentation was fixed at 2 spaces and not flexible. I realized I needed to allow users to adjust the indentation size, so I added a counter for this purpose.

Handling large JSON files was another challenge. Initially, synchronous processing caused the app to freeze with big inputs. I switched to asynchronous processing and added a loading state to improve performance.

The copy-to-clipboard feature also needed refinement. Users wanted feedback when copying succeeded or failed, so I added toast notifications for better UX.

### 2.2 Temporary Solutions

1. **JSON Parsing Issue**
   - Temporary: Directly use `JSON.parse` (simple but risky)
   - Problem: Crashes on invalid JSON
   - Final: Add try-catch and user-friendly error messages

2. **Indentation Setting Issue**
   - Temporary: Fixed at 2 spaces
   - Problem: Not flexible
   - Final: Add a counter for user adjustment

3. **Large JSON Handling**
   - Temporary: Synchronous processing
   - Problem: Freezes on large JSON
   - Final: Asynchronous processing with loading state

4. **Copy Function**
   - Temporary: Direct copy
   - Problem: No feedback
   - Final: Add success/failure notifications

### 2.3 Debugging Examples

1. **JSON Formatting**
```typescript
private async formatJson() {
  if (!this.inputJson) {
    await promptAction.showToast({ message: 'Please enter JSON text' })
    return
  }
  try {
    this.isProcessing = true
    const parsed: object = JSON.parse(this.inputJson)
    this.formattedJson = JSON.stringify(parsed, null, this.indentSize)
    await promptAction.showToast({ message: 'Formatting successful' })
  } catch (err) {
    console.error('JSON formatting failed:', err)
    await promptAction.showToast({ message: 'JSON format error' })
  } finally {
    this.isProcessing = false
  }
}
```

2. **Copy to Clipboard**
```typescript
private async copyToClipboard(): Promise<void> {
  try {
    const pasteData: pasteboard.PasteData = pasteboard.createData(
      pasteboard.MIMETYPE_TEXT_PLAIN,
      this.formattedJson
    )
    const systemPasteboard = pasteboard.getSystemPasteboard()
    await systemPasteboard.setData(pasteData)
    await promptAction.showToast({ message: 'Copied to clipboard' })
  } catch (err) {
    console.error('Copy failed:', err)
    await promptAction.showToast({ message: 'Copy failed' })
  }
}
```

## III. Pitfalls and Lessons

### 3.1 Problems Encountered

1. **JSON Parsing**
   - Problem: Crashes on invalid JSON
   - Solution: Add try-catch
   - Suggestion: Provide user-friendly error messages

2. **Indentation Setting**
   - Problem: Not flexible
   - Solution: Add a counter
   - Suggestion: Let users adjust indentation

3. **Large JSON Handling**
   - Problem: Freezes on large JSON
   - Solution: Asynchronous processing
   - Suggestion: Add loading state

4. **Copy Function**
   - Problem: No feedback
   - Solution: Add notifications
   - Suggestion: Handle errors gracefully

### 3.2 Optimization Suggestions

1. **Feature Optimization**
   - Support more formats
   - Add history records
   - Support batch formatting
   - Add format classification, import/export, management, sharing, backup, etc.

2. **Performance Optimization**
   - Optimize parsing speed
   - Reduce memory usage
   - Release resources promptly
   - Try multithreading, algorithm optimization, result caching, asynchronous processing, etc.

3. **User Experience**
   - Add usage instructions
   - Support keyboard shortcuts
   - Add animation effects, themes, sharing, favorites, import, backup, etc.

## IV. Summary

The JSON Formatter tool now covers all basic functions:

- JSON formatting
- Real-time result preview
- One-click copy
- Favorites for common settings

Some minor issues remain, but it works well for most scenarios. Further optimizations will be made over time.

## V. Reference Resources
- [HarmonyOS Application Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## Welcome to Experience

This JSON Formatter tool is integrated into the HarmonyOS Developer Toolbox. Welcome to download and try it!

[HarmonyOS Developer Toolbox](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> Author: In the World of Development
> Email: 1743914721@qq.com
> Copyright Notice: This article is original. Please indicate the source when reprinting.

---

If you encounter similar problems, feel free to leave a comment and discuss. If you can't solve it, let's have a headache together! 