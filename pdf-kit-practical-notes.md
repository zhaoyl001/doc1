# HarmonyOS PDF Kit Practical Notes


## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) represents a revolutionary step in the evolution of Huawei's distributed operating system. As someone who has been following its development closely, I can attest to the remarkable improvements in this version. The system's microkernel architecture not only enhances security but also provides unprecedented flexibility in cross-device collaboration. What excites me most about HarmonyOS 5 is its focus on developer experience - the new ArkTS language, enhanced UI components, and improved debugging tools have made development much more efficient. The distributed capabilities allow us to create truly seamless experiences across different devices, from smartphones to tablets, smart TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer based in China, with over three years of experience in the ecosystem. My journey with HarmonyOS began with its early versions, and I've witnessed its remarkable evolution. As a developer who has contributed to several open-source HarmonyOS projects, I've gained valuable insights into its architecture and best practices. I'm particularly passionate about real-time communication and distributed applications, which led me to explore WebSocket implementations in HarmonyOS 5. Through my blog and GitHub repositories, I've been sharing my experiences and helping other developers navigate the exciting world of HarmonyOS development.

## Why This Note Exists
PDFs are everywhere in dev work. My first time dealing with PDF features was when a colleague asked me to help with a contract preview. I was totally lost and hit a bunch of snags. But after using HarmonyOS PDF Kit for a while, I found it pretty handy—editing, previewing, adding annotations, you name it. Most dev needs are covered.

This note is meant as a "pitfall guide" for those who come after me, and a record of the little traps and lessons I picked up. If you use PDF Kit, I hope you'll take fewer detours and have a bit more fun.

## What's This Kit Anyway?

HarmonyOS PDF Kit gives you a bunch of PDF document handling features, with two main modules: `pdfService` and `PdfView`.

- **pdfService**: For loading, saving, and editing PDFs—add text, images, annotations, headers/footers, watermarks, backgrounds, bookmarks, encryption/decryption, and more.
- **PdfView**: For previewing PDFs, page navigation, zooming, keyword search, highlighting, annotations, etc.

Sometimes, all it takes is a product manager saying, "Can we add PDF annotations?" and you have to dig through the API from scratch. Don't panic—these examples and stories are all from my real-world experience.

For more examples, check out the official CodeLab and SampleCode (or just poke around and see what breaks).

## What Can Each Module Do? (A Not-So-Official Comparison)

Let's be honest: pdfService and PdfView each have their own superpowers. Here's my not-so-official, developer-style rundown:

- Opening and saving documents? Both can do it, no sweat.
- Releasing documents? Both handle it, so no memory leaks to worry about.
- PDF to image? Both can, though I rarely use it.
- Annotations? Both can add and remove them—whatever the product wants.
- Bookmarks? pdfService can manage them, PdfView can't.
- Adding/removing PDF pages, text, images, watermarks, headers/footers? pdfService does it all; PdfView just sticks to previewing.
- Checking if a PDF is encrypted or decrypting? pdfService can do it, PdfView just watches.
- Preview, search, event callbacks? That's PdfView's turf—pdfService doesn't get involved.

Bottom line: pdfService is the "hands-on" type—edit, add, delete, whatever. PdfView is more of a "viewer"—great for previewing, flipping pages, searching, and annotating. In real development, just use whichever feels right. Don't get stuck in the API docs—try, fail, and you'll get it.

> Sometimes I wish pdfService and PdfView could merge, so I wouldn't have to switch back and forth. But for now, they each do their own thing—just roll with it.

## Some Annoying Limits
- **Supported regions**: Mainland China only (not including Hong Kong, Macau, or Taiwan).
- **Supported devices**: Real devices only (Phone, Tablet, PC/2in1). Emulators are not supported.

---

## Opening and Saving PDF Documents (How I Usually Do It)

- For editing PDF content, use `pdfService`.
- For preview, search, or event listening, use `PdfView`.

What I usually call:
- `loadDocument(path: string, password?: string, onProgress?: Callback<number>): ParseResult` — Load a PDF.
- `saveDocument(path: string, onProgress?: Callback<number>): boolean` — Save a PDF.

A little story:
The first time I did a "Save As" feature, I got nothing when I clicked the button. Turns out, I mixed up the sandbox and resource paths—don't try to write a PDF to a read-only directory!

Here's how I did it (and it finally worked):
```typescript
import { pdfService } from '@kit.PDFKit';
import { hilog } from '@kit.PerformanceAnalysisKit';
import { fileIo } from '@kit.CoreFileKit';

@Entry
@Component
struct PdfPage {
  private pdfDocument: pdfService.PdfDocument = new pdfService.PdfDocument();
  private context = this.getUIContext().getHostContext() as Context;
  private filePath = '';
  @State saveEnable: boolean = false;

  aboutToAppear(): void {
    this.filePath = this.context.filesDir + '/input.pdf';
    let res = fileIo.accessSync(this.filePath);
    if(!res) {
      // Make sure input.pdf exists in src/main/resources/rawfile
      let content: Uint8Array = this.context.resourceManager.getRawFileContentSync('rawfile/input.pdf');
      let fdSand = fileIo.openSync(this.filePath, fileIo.OpenMode.WRITE_ONLY | fileIo.OpenMode.CREATE | fileIo.OpenMode.TRUNC);
      fileIo.writeSync(fdSand.fd, content.buffer);
      fileIo.closeSync(fdSand.fd);
    }
    this.pdfDocument.loadDocument(this.filePath);
  }

  build() {
    Column() {
      // Save as a new PDF
      Button('Save As').onClick(() => {
        let outPdfPath = this.context.filesDir + '/testSaveAsPdf.pdf';
        let result = this.pdfDocument.saveDocument(outPdfPath);
        this.saveEnable = true;
        hilog.info(0x0000, 'PdfPage', 'saveAsPdf %{public}s!', result ? 'success' : 'fail');
      })
      // Overwrite original PDF
      Button('Save').enabled(this.saveEnable).onClick(() => {
        let tempDir = this.context.tempDir;
        let tempFilePath = tempDir + `/temp${Math.random()}.pdf`;
        fileIo.copyFileSync(this.filePath, tempFilePath);
        let pdfDocument: pdfService.PdfDocument = new pdfService.PdfDocument();
        let loadResult = pdfDocument.loadDocument(tempFilePath, '');
        if (loadResult === pdfService.ParseResult.PARSE_SUCCESS) {
          let result = pdfDocument.saveDocument(this.filePath);
          hilog.info(0x0000, 'PdfPage', 'savePdf %{public}s!', result ? 'success' : 'fail');
        }
      })
    }
  }
}
```

---

## Adding and Deleting PDF Pages (Yes, You'll Need This)

- Supports inserting blank pages, merging pages from other PDFs, and deleting specific pages.

What I usually call:
- `insertBlankPage(index, width, height)` — Insert a blank page.
- `getPage(index)` — Get a page object.
- `insertPageFromDocument(document, fromIndex, pageCount, index)` — Merge pages from another document.
- `deletePage(index, count)` — Delete pages.

A quick anecdote:
A tester once asked, "Why are all the inserted pages at the end?" Turns out, I misunderstood the index parameter. Get the insert position right, or the user experience will be weird.

Here's a code snippet (don't copy-paste blindly, tweak as needed):
```typescript
import { pdfService } from '@kit.PDFKit';
import { hilog } from '@kit.PerformanceAnalysisKit';

@Entry
@Component
struct PdfPage {
  private pdfDocument: pdfService.PdfDocument = new pdfService.PdfDocument();
  private context = this.getUIContext().getHostContext() as Context;

  aboutToAppear(): void {
    let filePath = this.context.filesDir + '/input.pdf';
    this.pdfDocument.loadDocument(filePath);
  }

  build() {
    Column() {
      // Insert a single blank page
      Button('insertBlankPage').onClick(async () => {
        let page = this.pdfDocument.getPage(0);
        this.pdfDocument.insertBlankPage(2, page.getWidth(), page.getHeight());
        let outPdfPath = this.context.filesDir + '/testInsertBlankPage.pdf';
        let result = this.pdfDocument.saveDocument(outPdfPath);
        hilog.info(0x0000, 'PdfPage', 'insertBlankPage %{public}s!', result ? 'success' : 'fail');
      })
      // Insert multiple blank pages
      Button('insertSomeBlankPage').onClick(async () => {
        let page = this.pdfDocument.getPage(0);
        for (let i = 0; i < 3; i++) {
          this.pdfDocument.insertBlankPage(2, page.getWidth(), page.getHeight());
        }
        let outPdfPath = this.context.filesDir + '/testInsertSomeBlankPage.pdf';
        let result = this.pdfDocument.saveDocument(outPdfPath);
        hilog.info(0x0000, 'PdfPage', 'insertSomeBlankPage %{public}s!', result ? 'success' : 'fail');
      })
      // Merge pages from another PDF
      Button('insertPageFromDocument').onClick(async () => {
        let pdfDoc = new pdfService.PdfDocument();
        pdfDoc.loadDocument(this.context.filesDir + '/input2.pdf');
        this.pdfDocument.insertPageFromDocument(pdfDoc, 1, 3, 0);
        let outPdfPath = this.context.filesDir + '/testInsertPageFromDocument.pdf';
        let result = this.pdfDocument.saveDocument(outPdfPath);
        hilog.info(0x0000, 'PdfPage', 'insertPageFromDocument %{public}s!', result ? 'success' : 'fail');
      })
      // Delete pages
      Button('deletePage').onClick(async () => {
        this.pdfDocument.deletePage(2, 2);
        let outPdfPath = this.context.filesDir + '/testDeletePage.pdf';
        let result = this.pdfDocument.saveDocument(outPdfPath);
        hilog.info(0x0000, 'PdfPage', 'deletePage %{public}s!', result ? 'success' : 'fail');
      })
    }
  }
}
```

---

## Previewing PDF Documents (Don't Skip This)

- Supports page navigation, zoom, single/double-page display, fit modes, scrolling, search, annotations, and more.
- Make sure the PDF file is in the sandbox directory.

Dev thoughts:
When previewing PDFs, the worst is "slow loading" or "laggy page turns." I recommend using event callbacks and adding a loading animation for users—it makes a big difference.

Here's what worked for me:
```typescript
import { pdfService, pdfViewManager, PdfView } from '@kit.PDFKit';
import { fileIo } from '@kit.CoreFileKit';
import { hilog } from '@kit.PerformanceAnalysisKit';

@Entry
@Component
struct Index {
  private controller: pdfViewManager.PdfController = new pdfViewManager.PdfController();

  aboutToAppear(): void {
    let context = this.getUIContext().getHostContext() as Context;
    let dir = context.filesDir;
    let filePath = dir + '/input.pdf';
    let res = fileIo.accessSync(filePath);
    if (!res) {
      let content = context.resourceManager.getRawFileContentSync('rawfile/input.pdf');
      let fdSand = fileIo.openSync(filePath, fileIo.OpenMode.WRITE_ONLY | fileIo.OpenMode.CREATE | fileIo.OpenMode.TRUNC);
      fileIo.writeSync(fdSand.fd, content.buffer);
      fileIo.closeSync(fdSand.fd);
    }
    (async () => {
      // Register listener before loading document
      this.controller.registerPageCountChangedListener((pageCount: number) => {
        hilog.info(0x0000, 'registerPageCountChanged-', pageCount.toString());
      });
      let loadResult1 = await this.controller.loadDocument(filePath);
    })();
  }

  build() {
    Row() {
      PdfView({
        controller: this.controller,
        pageFit: pdfService.PageFit.FIT_WIDTH,
        showScroll: true
      })
        .id('pdfview_app_view')
        .layoutWeight(1);
    }
    .width('100%')
    .height('100%')
  }
}
```

---

## Advanced PdfView Usage (A Few More Tricks)

### Asynchronous Opening and Saving (Promise Style)

A little story:
Once, I had a huge file and the UI just froze when saving. Later, I realized I should use Promises for async operations—don't block the main thread, and the user experience gets way better.

Here's the async way:
```typescript
import { pdfService, PdfView, pdfViewManager } from '@kit.PDFKit';
import { hilog } from '@kit.PerformanceAnalysisKit';

@Entry
@Component
struct PdfPage {
  private controller: pdfViewManager.PdfController = new pdfViewManager.PdfController();
  private context = this.getUIContext().getHostContext() as Context;
  private loadResult: pdfService.ParseResult = pdfService.ParseResult.PARSE_ERROR_FORMAT;

  aboutToAppear(): void {
    let filePath = this.context.filesDir + '/input.pdf';
    (async () => {
      this.loadResult = await this.controller.loadDocument(filePath);
    })()
  }

  build() {
    Column() {
      Button('savePdfDocument').onClick(async () => {
        if (this.loadResult === pdfService.ParseResult.PARSE_SUCCESS) {
          let savePath = this.context.filesDir + '/savePdfDocument.pdf';
          let result = await this.controller.saveDocument(savePath);
          hilog.info(0x0000, 'PdfPage', 'savePdfDocument %{public}s!', result ? 'success' : 'fail');
        }
      })
      PdfView({
        controller: this.controller,
        pageFit: pdfService.PageFit.FIT_WIDTH,
        showScroll: true
      })
        .id('pdfview_app_view')
        .layoutWeight(1);
    }
    .width('100%')
    .height('100%')
  }
}
```

---

### Setting PDF Preview Effects (Because PMs Love Fancy Stuff)

Dev anecdote:
The product manager said, "Can we do double-page like a book?" I thought it'd be tough, but it was just one line with setPageLayout. Sometimes HarmonyOS APIs are surprisingly friendly.

Try this (and impress your PM):
```typescript
import { pdfService, PdfView, pdfViewManager } from '@kit.PDFKit';

@Entry
@Component
struct PdfPage {
  private controller: pdfViewManager.PdfController = new pdfViewManager.PdfController();
  private context = this.getUIContext().getHostContext() as Context;
  private loadResult: pdfService.ParseResult = pdfService.ParseResult.PARSE_ERROR_FORMAT;

  aboutToAppear(): void {
    let filePath = this.context.filesDir + '/input.pdf';
    (async () => {
      this.loadResult = await this.controller.loadDocument(filePath);
    })()
  }

  build() {
    Column() {
      Row() {
        Button('setPreviewMode').onClick(() => {
          if (this.loadResult === pdfService.ParseResult.PARSE_SUCCESS) {
            this.controller.setPageLayout(pdfService.PageLayout.LAYOUT_SINGLE); // Single page
            this.controller.setPageContinuous(true); // Continuous scroll
            this.controller.setPageFit(pdfService.PageFit.FIT_PAGE); // Fit whole page
          }
        })
        Button('goToPage').onClick(() => {
          if (this.loadResult === pdfService.ParseResult.PARSE_SUCCESS) {
            this.controller.goToPage(10); // Jump to page 11
          }
        })
        Button('zoomPage2').onClick(() => {
          if (this.loadResult === pdfService.ParseResult.PARSE_SUCCESS) {
            this.controller.setPageZoom(2); // Zoom in 2x
          }
        })
      }
      PdfView({
        controller: this.controller,
        pageFit: pdfService.PageFit.FIT_WIDTH,
        showScroll: true
      })
        .id('pdfview_app_view')
        .layoutWeight(1);
    }
  }
}
```

---

### Keyword Search and Highlighting (For the Perfectionists)

- Supports searching PDF content, auto-highlighting all matches.
- Jump to previous/next match, get the current highlight index, clear all highlights.

A quick story:
A user once said, "Why doesn't searching for C++ work?" Turns out, you have to watch out for case and special characters. The API isn't case-sensitive, but some symbols need escaping.

Here's how I handle it:
```typescript
import { pdfService, PdfView, pdfViewManager } from '@kit.PDFKit';
import { hilog } from '@kit.PerformanceAnalysisKit';

@Entry
@Component
struct PdfPage {
  private controller: pdfViewManager.PdfController = new pdfViewManager.PdfController();
  private context = this.getUIContext().getHostContext() as Context;
  private loadResult: pdfService.ParseResult = pdfService.ParseResult.PARSE_ERROR_FORMAT;
  private searchIndex = 0;
  private charCount = 0;

  aboutToAppear(): void {
    let filePath = this.context.filesDir + '/input.pdf';
    (async () => {
      this.loadResult = await this.controller.loadDocument(filePath);
    })()
  }

  build() {
    Column() {
      Row() {
        Button('searchKey').onClick(async () => {
          if (this.loadResult === pdfService.ParseResult.PARSE_SUCCESS) {
            this.controller.searchKey('C++', (index: number) => {
              this.charCount = index;
              hilog.info(0x0000, 'PdfPage', 'searchKey %{public}s!', index + '');
            })
          }
        })
        Button('setSearchPrevIndex').onClick(async () => {
          if (this.loadResult === pdfService.ParseResult.PARSE_SUCCESS) {
            if(this.searchIndex > 0) {
              this.controller.setSearchIndex(--this.searchIndex);
            }
          }
        })
        Button('setSearchNextIndex').onClick(async () => {
          if (this.loadResult === pdfService.ParseResult.PARSE_SUCCESS) {
            if(this.searchIndex < this.charCount) {
              this.controller.setSearchIndex(++this.searchIndex);
            }
          }
        })
        Button('getSearchIndex').onClick(async () => {
          if (this.loadResult === pdfService.ParseResult.PARSE_SUCCESS) {
            let curSearchIndex = this.controller.getSearchIndex();
            hilog.info(0x0000, 'PdfPage', 'curSearchIndex %{public}s!', curSearchIndex + '');
          }
        })
        Button('clearSearch').onClick(async () => {
          if (this.loadResult === pdfService.ParseResult.PARSE_SUCCESS) {
            this.controller.clearSearch();
          }
        })
      }
      PdfView({
        controller: this.controller,
        pageFit: pdfService.PageFit.FIT_WIDTH,
        showScroll: true
      })
        .id('pdfview_app_view')
        .layoutWeight(1);
    }
  }
}
```

---

## Gotchas & Tips
- Only supports real devices in mainland China; emulators and Hong Kong/Macau/Taiwan are not supported.
- Resource files must be placed in the rawfile directory and copied to the sandbox.
- For editing, use pdfService; for pure preview, use PdfView.
- Be careful with file paths and permissions when saving/overwriting.
- If you get stuck, don't panic—sometimes even the docs won't help, but trial and error usually does. And if you find a better way, let me know!

---

## Where to Find More (or Get Stuck)
- [PDF Kit Official Documentation](https://developer.harmonyos.com/cn/docs/documentation/doc-references/pdfkit-overview-0000001554328913)
- [HarmonyOS Developer Community](https://developer.harmonyos.com/cn/community) 