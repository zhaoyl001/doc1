# HarmonyOS Developer Toolbox - Amount Converter Development Notes

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) is a major milestone in Huawei's distributed operating system. Its microkernel architecture enhances security and enables seamless cross-device collaboration. The developer experience is greatly improved with the new ArkTS language, enhanced UI components, and better debugging tools. Distributed capabilities allow for smooth experiences across smartphones, tablets, TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer from China, with years of experience in the HarmonyOS ecosystem. I focus on cross-platform development and performance optimization, and I enjoy sharing my technical insights and practical experiences with the community.

## Basic Knowledge

Before diving into the Amount Converter, let's briefly review some basic concepts:

- **Amount Conversion:** The process of converting numeric amounts into Chinese uppercase representations, e.g., "1234.56" to "壹仟贰佰叁拾肆元伍角陆分".
- **Decimal Handling:** Managing the number of decimal places, rounding, and formatting.
- **Zero Handling:** Special rules for consecutive zeros and leading zeros in Chinese uppercase conversion.

## Preface
Recently, while working on the HarmonyOS Toolbox, I wanted to add an amount conversion feature. At first glance, it seemed simple, but once I started, I realized there were many pitfalls. For example, converting "1234.56" to "壹仟贰佰叁拾肆元伍角陆分" involved handling edge cases, and it took several days of debugging to get it right.

During development, I encountered many issues, such as amount formatting, decimal places, and zero handling. In the end, all problems were solved, and the tool is now quite handy.

## I. Feature Overview

### 1.1 Main Features
- Convert numbers to Chinese uppercase
- Support for decimals
- One-click copy
- Favorites feature

### 1.2 UI Features
- Amount input box
- Convert button
- Result display area
- Copy button

## II. Implementation Process

### 2.1 Development Stories

At first, I copied a conversion function from the internet, but it had many issues. For example, entering "1000.00" produced "壹仟元整", but users complained it should be "壹仟元零角零分". I found that different scenarios have different requirements for zeros—sometimes they should be shown, sometimes not.

Another time, a user wanted "0.1" to convert to "零元壹角" instead of just "壹角". I revised the code several times and finally added logic to display "零元" when the integer part is 0.

The most confusing was handling decimals. Initially, there was no limit on decimal places, so "123.456" became "壹佰贰拾叁元肆角伍分陆厘", which confused users. I then limited the output to two decimal places.

### 2.2 Temporary Solutions

1. **Zero Handling**
   - Temporary: Use regex to replace consecutive zeros with a single zero
   - Problem: Sometimes removes zeros that should be kept, causing strange results
   - Final: Use a counter to track consecutive zeros for reliable handling

2. **Large Number Handling**
   - Temporary: String concatenation, adding one digit at a time
   - Problem: Slow and error-prone, especially for large numbers
   - Final: Process in segments of four digits for higher efficiency

3. **Decimal Handling**
   - Temporary: Truncate to two decimal places
   - Problem: Sometimes loses precision, users dissatisfied
   - Final: Round first, then truncate for better experience

4. **Leading Zero Handling**
   - Temporary: Remove all leading zeros
   - Problem: Users wanted to keep one zero for "00123"
   - Final: Keep one zero if all digits are zero, otherwise remove

### 2.3 Debugging Examples

1. **Amount Formatting**
```typescript
private formatAmount(value: string): string {
  // Remove non-numeric and non-dot characters
  value = value.replace(/[^\d.]/g, '');
  
  // Handle multiple dots
  const parts = value.split('.');
  if (parts.length > 2) {
    value = parts[0] + '.' + parts[1];
  }

  // Handle decimal places
  if (parts.length === 2) {
    parts[1] = parts[1].slice(0, 2);
    value = parts[0] + (parts[1] ? '.' + parts[1] : '');
  }

  // Handle leading zeros
  if (parts[0].length > 1 && parts[0].startsWith('0')) {
    parts[0] = parts[0].replace(/^0+/, '') || '0';
    value = parts[0] + (parts.length > 1 ? '.' + parts[1] : '');
  }

  // If only a dot is entered, add leading zero
  if (value === '.') {
    value = '0.';
  }

  return value;
}
```

2. **Amount Validation**
```typescript
private validateAmount(amount: string): boolean {
  this.showError = false;
  this.errorMessage = '';

  if (!amount) {
    this.showError = true;
    this.errorMessage = 'Please enter an amount';
    return false;
  }

  if (!/^\d+(\.\d{0,2})?$/.test(amount)) {
    this.showError = true;
    this.errorMessage = 'Please enter a valid amount format';
    return false;
  }

  const num = parseFloat(amount);

  if (num > this.MAX_AMOUNT) {
    this.showError = true;
    this.errorMessage = 'Amount exceeds the limit';
    return false;
  }

  if (num < this.MIN_AMOUNT) {
    this.showError = true;
    this.errorMessage = 'Amount cannot be less than 0.01';
    return false;
  }

  return true;
}
```

3. **Conversion Implementation**
```typescript
static numberToChineseMoney(money: string): string {
  const num = parseFloat(money);
  if (isNaN(num)) return 'Invalid amount format';
  if (num === 0) return '零元整';

  const cnNums = ['零', '壹', '贰', '叁', '肆', '伍', '陆', '柒', '捌', '玖'];
  const cnIntRadice = ['', '拾', '佰', '仟'];
  const cnIntUnits = ['', '万', '亿', '兆'];
  const cnDecUnits = ['角', '分'];
  
  const parts = money.split('.');
  let integerPart = parts[0];
  const decimalPart = parts[1] ? parts[1].substr(0, 2) : '';
  
  let chineseStr = '';
  
  // Process integer part
  if (parseInt(integerPart) > 0) {
    let zeroCount = 0;
    const IntLen = integerPart.length;
    for (let i = 0; i < IntLen; i++) {
      const n = parseInt(integerPart.substr(i, 1));
      const p = IntLen - i - 1;
      const q = p / 4;
      const m = p % 4;
      
      if (n === 0) {
        zeroCount++;
      } else {
        if (zeroCount > 0) {
          chineseStr += cnNums[0];
        }
        zeroCount = 0;
        chineseStr += cnNums[n] + cnIntRadice[m];
      }
      
      if (m === 0 && zeroCount < 4) {
        chineseStr += cnIntUnits[q];
      }
    }
    chineseStr += '元';
  }

  // Process decimal part
  if (decimalPart) {
    for (let i = 0; i < decimalPart.length; i++) {
      const n = parseInt(decimalPart.substr(i, 1));
      if (n !== 0) {
        chineseStr += cnNums[n] + cnDecUnits[i];
      }
    }
  }

  if (!decimalPart) {
    chineseStr += '整';
  }

  return chineseStr;
}
```

## III. Pitfalls and Lessons

### 3.1 Problems Encountered

1. **Amount Format Issues**
   - Problem: Users enter irregular formats, e.g., multiple dots
   - Solution: Add formatting function to auto-handle
   - Suggestion: Restrict format at input to avoid trouble

2. **Decimal Places Issues**
   - Problem: Decimal places not fixed
   - Solution: Always keep two decimal places
   - Suggestion: Allow user customization if needed

3. **Zero Handling Issues**
   - Problem: Consecutive zeros not handled properly
   - Solution: Keep only one zero for multiple zeros
   - Suggestion: Add custom options if needed

4. **Amount Range Issues**
   - Problem: Amount out of range
   - Solution: Add range limits
   - Suggestion: Allow custom range for flexibility

### 3.2 Optimization Suggestions

1. **Feature Optimization**
   - Support more amount formats
   - Add history records
   - Support batch conversion
   - Add amount classification, import/export, management, sharing, backup, etc.

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

The amount converter tool now covers all basic functions:

- Convert numbers to Chinese uppercase
- Support for decimal conversion
- One-click copy of results
- Favorites for common settings

Some minor issues remain, but it works well for most scenarios. Further optimizations will be made over time.

## V. Reference Resources
- [HarmonyOS Application Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## Welcome to Experience

This amount converter tool is integrated into the HarmonyOS Developer Toolbox. Welcome to download and try it!

[HarmonyOS Developer Toolbox](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> Author: In the World of Development
> Email: 1743914721@qq.com
> Copyright Notice: This article is original. Please indicate the source when reprinting.

---

If you encounter similar problems, feel free to leave a comment and discuss. If you can't solve it, let's have a headache together! 