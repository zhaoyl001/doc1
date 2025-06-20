# HarmonyOS Developer Toolbox - Color Converter Development Notes

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) is a major milestone in Huawei's distributed operating system. Its microkernel architecture enhances security and enables seamless cross-device collaboration. The developer experience is greatly improved with the new ArkTS language, enhanced UI components, and better debugging tools. Distributed capabilities allow for smooth experiences across smartphones, tablets, TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer from China, with years of experience in the HarmonyOS ecosystem. I focus on cross-platform development and performance optimization, and I enjoy sharing my technical insights and practical experiences with the community.

## Basic Knowledge

Before diving into the Color Converter, let's briefly review some basic concepts:

- **HEX:** A hexadecimal color code, e.g., #FF0000.
- **RGB:** Red, Green, Blue color model, e.g., rgb(255, 0, 0).
- **HSL:** Hue, Saturation, Lightness color model, e.g., hsl(0, 100%, 50%).
- **Color Space Conversion:** The process of converting between different color representations.

## Preface
Recently, while working on the HarmonyOS Toolbox, I wanted to add a color conversion feature. The main purpose is to convert between various color formats, such as converting "#FF0000" to "rgb(255, 0, 0)" or "hsl(0, 100%, 50%)". At first, I thought it would be simple, but I soon realized that handling all the formats was tricky and required several rounds of debugging.

During development, I encountered many pitfalls, such as color format validation, color space conversion, and implementing a color picker. In the end, all issues were resolved, and the tool is now quite handy.

## I. Feature Overview

### 1.1 Main Features
- Support conversion between multiple color formats: HEX, RGB, HSL
- Visual color picker
- Quick selection of common colors
- Precise adjustment of HSL parameters
- Real-time preview of conversion results
- One-click copy of results
- Favorites feature

### 1.2 UI Features
- Text input box
- Color picker
- Format selection list
- Result display area
- Copy button

## II. Implementation Process

### 2.1 Development Stories

At first, I used a conversion function found online, but it had many issues. For example, when entering "#FFF", the output was "rgb(255, 255, 255)", but users said it should be "rgb(255, 255, 255, 1)". I found that different scenarios require different handling of color formats, sometimes with alpha, sometimes without.

Another time, a user wanted to see "hsl(0, 100%, 50%)" when converting "rgb(255, 0, 0)" to "#FF0000". This requirement made me revise the code several times, and I finally added logic to convert based on the selected format.

The funniest was handling HSL conversion. Initially, I didn't support HSL, but users insisted on it. After adding HSL conversion, I found that converting HSL to RGB sometimes produced errors, such as out-of-range hue values. After several revisions, it finally worked.

There was also a small episode where a user said the color picker was not easy to use. At first, I thought it was simple, but after trying it myself, I found issues such as selecting the wrong color and the preview not being intuitive. After several iterations, I added a magnifier effect, which improved the experience.

### 2.2 Temporary Solutions

1. **Color Format Validation**
   - Temporary: Use regex for validation (simple but crude)
   - Problem: Sometimes validates incorrect formats, causing errors
   - Final: Use specific regex for each format for reliability

2. **Color Space Conversion**
   - Temporary: Convert directly to RGB, ignore other formats
   - Problem: Sometimes converts formats that shouldn't be converted, causing user dissatisfaction
   - Final: Determine whether conversion is needed based on format type

3. **Color Picker**
   - Temporary: Use system color picker
   - Problem: Users said it was not user-friendly, wanted customization
   - Final: Implement a custom color picker for better experience

4. **HSL Adjustment**
   - Temporary: Input HSL values in a text box
   - Problem: Users said it was not user-friendly, wanted sliders
   - Final: Use sliders for HSL adjustment, improving user experience

### 2.3 Debugging Examples

1. **Color Format Validation**
```typescript
static isValidHex(color: string): boolean {
  return /^#([A-Fa-f0-9]{3}){1,2}$/.test(color)
}

static isValidRGB(color: string): boolean {
  return /^rgb\(\s*(\d{1,3})\s*,\s*(\d{1,3})\s*,\s*(\d{1,3})\s*\)$/.test(color)
}

static isValidHSL(color: string): boolean {
  return /^hsl\(\s*(\d{1,3})\s*,\s*(\d{1,3})%\s*,\s*(\d{1,3})%\s*\)$/.test(color)
}
```

2. **Color Space Conversion**
```typescript
static hexToRgb(hex: string): string {
  hex = hex.replace('#', '')
  if(hex.length === 3) {
    hex = hex.split('').map(char => char + char).join('')
  }
  const r: number = parseInt(hex.substring(0, 2), 16)
  const g: number = parseInt(hex.substring(2, 4), 16)
  const b: number = parseInt(hex.substring(4, 6), 16)
  return `rgb(${r}, ${g}, ${b})`
}

static rgbToHex(rgb: string): string {
  const matches: RegExpMatchArray | null = rgb.match(/^rgb\((\d+),\s*(\d+),\s*(\d+)\)$/)
  if (!matches) return rgb
  const r: number = parseInt(matches[1])
  const g: number = parseInt(matches[2])
  const b: number = parseInt(matches[3])
  const toHex = (n: number): string => {
    const hex = n.toString(16)
    return hex.length === 1 ? '0' + hex : hex
  }
  return `#${toHex(r)}${toHex(g)}${toHex(b)}`
}
```

## III. Pitfalls and Lessons

### 3.1 Problems Encountered

1. **Color Format Validation**
   - Problem: Users enter irregular formats, e.g., multiple separators
   - Solution: Add regex validation, auto-handle
   - Suggestion: Restrict format at input to avoid trouble

2. **Color Space Conversion**
   - Problem: Inaccurate conversion results
   - Solution: Determine by format type
   - Suggestion: Allow user customization

3. **Color Picker**
   - Problem: Picker not user-friendly
   - Solution: Implement custom picker
   - Suggestion: Allow custom pickers

4. **HSL Adjustment**
   - Problem: Adjustment not convenient
   - Solution: Use sliders
   - Suggestion: Allow custom adjustment methods

### 3.2 Optimization Suggestions

1. **Feature Optimization**
   - Support more formats
   - Add history records
   - Support batch conversion
   - Add format classification, import/export, management, sharing, backup, etc.

2. **Performance Optimization**
   - Optimize conversion speed
   - Reduce memory usage
   - Release resources promptly
   - Try multithreading, algorithm optimization, result caching, asynchronous processing, etc.

3. **User Experience**
   - Add usage instructions
   - Support keyboard shortcuts
   - Add animation effects, themes, sharing, favorites, import, backup, etc.

## IV. Summary

The color converter tool now covers all basic functions:

- Support for multiple format conversions
- Real-time result preview
- One-click copy of results
- Favorites for common settings

Some minor issues remain, but it works well for most scenarios. Further optimizations will be made over time.

## V. Reference Resources
- [HarmonyOS Application Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## Welcome to Experience

This color converter tool is integrated into the HarmonyOS Developer Toolbox. Welcome to download and try it!

[HarmonyOS Developer Toolbox](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> Author: In the World of Development
> Email: 1743914721@qq.com
> Copyright Notice: This article is original. Please indicate the source when reprinting.

---

If you encounter similar problems, feel free to leave a comment and discuss. If you can't solve it, let's have a headache together! 