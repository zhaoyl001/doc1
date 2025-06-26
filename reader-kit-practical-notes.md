# Reader Kit Practical Notes (HarmonyOS)


## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) represents a revolutionary step in the evolution of Huawei's distributed operating system. As someone who has been following its development closely, I can attest to the remarkable improvements in this version. The system's microkernel architecture not only enhances security but also provides unprecedented flexibility in cross-device collaboration. What excites me most about HarmonyOS 5 is its focus on developer experience - the new ArkTS language, enhanced UI components, and improved debugging tools have made development much more efficient. The distributed capabilities allow us to create truly seamless experiences across different devices, from smartphones to tablets, smart TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer based in China, with over three years of experience in the ecosystem. My journey with HarmonyOS began with its early versions, and I've witnessed its remarkable evolution. As a developer who has contributed to several open-source HarmonyOS projects, I've gained valuable insights into its architecture and best practices. I'm particularly passionate about real-time communication and distributed applications, which led me to explore WebSocket implementations in HarmonyOS 5. Through my blog and GitHub repositories, I've been sharing my experiences and helping other developers navigate the exciting world of HarmonyOS development.

## So, What Can Reader Kit Actually Do?

The first time I tried to build an eBook reader, I thought it was just "read a txt and show it." Turns out, there are a million gotchas—formats, layout, page-flip effects, TOC jumps... all sorts of traps. Reader Kit honestly saved my bacon. This doc is just my "war story" for the next dev, and a reminder to myself.

### What's It Good At?
- Handles txt, epub, mobi, azw, azw3—title, author, cover, TOC, content, you name it.
- Lays out txt and rich text (html+css), does simulation and horizontal swipe paging, and even gives you layout snapshots.
- The ReadPageComponent shows content, handles page turns, animations, and tracks reading progress.

### Stuff I Actually Like
- Multi-format parsing—pretty much any book, any info, you can grab it.
- Rich text layout is fast, supports custom fonts, W3C standard, no compatibility headaches.
- Page-flip effects are smooth (thanks OpenGL), sometimes I just keep flipping for fun.

## Some Terms You Can't Avoid
- **ReadPageComponent**: The main UI for reading—shows content, handles page turns, tracks progress.
- **BookParser**: The engine that chews through any book format.
- **spine**: Not your back, but the book's reading order. Each SpineItem is a chunk of content.

## Gotchas & Limits
- Only reads local files—don't even think about cloud books (yet).
- DRM-protected books? Don't bother, they won't open.
- Only supports txt, epub, mobi, azw, azw3. Don't try pdf.
- Layout and interaction must use ReadPageComponent—rolling your own is a pain.
- Only works on real HarmonyOS NEXT 5.0.4+ devices. Emulators? Forget it.
- Mainland China only for now—HK, Macau, overseas, sorry.

---

## How Do I Get the Title and Cover?

Sometimes the product just wants to show a cover and title on the bookshelf. Reader Kit makes this easy.

Here's how I do it (don't mess up spineIndex, or you'll get a blank cover—don't ask how I know):

```typescript
// This code works if you copy-paste, just mind your paths and permissions
import { bookParser } from '@kit.ReaderKit';
import { image } from '@kit.ImageKit';

let path: string = './download/ebook/abc.epub';
let bookParserHandler: bookParser.BookParserHandler = await bookParser.getDefaultHandler(path);
let bookInfo: bookParser.BookInfo = bookParserHandler.getBookInfo();
let bookTitle = bookInfo.bookTitle;
let buffer: ArrayBuffer = bookParserHandler.getResourceContent(-1, bookInfo.bookCoverImage);
let imageSource: image.ImageSource = image.createImageSource(buffer);
let bookCover: image.PixelMap = await imageSource.createPixelMap();
imageSource.release();
```

---

## How Do I Handle TOC and Jumps?

TOC and chapter jumps—product always wants them. Reader Kit's TOC parsing and jumping is pretty smooth.

Here's how I do it (don't mess up catalogLevel or your TOC will look like stairs):

```typescript
// TOC rendering and jumping—copy this, but tweak the indentation
import { bookParser } from '@kit.ReaderKit';
import { hilog } from '@kit.PerformanceAnalysisKit';

@State catalogItemList: bookParser.CatalogItem[] = [];
private defaultHandler: bookParser.BookParserHandler | null = null;

private async getCatalogItemList(){
  let path: string = './download/ebook/abc.epub';
  this.defaultHandler = await bookParser.getDefaultHandler(path);
  this.catalogItemList = this.defaultHandler.getCatalogList() || [];
}

// Render TOC
build() {
  Column() {
    List() {
      ForEach(this.catalogItemList, (item: bookParser.CatalogItem) => {
        ListItem() {
          // ...chapter name, click to jump, etc...
        }
      })
    }
  }
}
```
Jumping to a chapter:
```typescript
private async jumpToCatalogItem(catalogItem: bookParser.CatalogItem){
  const domPos = await this.getDomPos(catalogItem);
  const resourceIndex = this.getResourceItemByCatalog(catalogItem).index;
  // Use domPos and resourceIndex to jump
}

private async getDomPos(catalogItem: bookParser.CatalogItem): Promise<string> {
  const domPos: string = this.defaultHandler?.getDomPosByCatalogHref(catalogItem.href || '') || '';
  return domPos;
}

private getResourceItemByCatalog(catalogItem: bookParser.CatalogItem): bookParser.SpineItem {
  let resourceFile = catalogItem.resourceFile || '';
  let spineList: bookParser.SpineItem[] = this.defaultHandler?.getSpineList() || [];
  let resourceItemArr = spineList.filter(item => item.href === resourceFile);
  if (resourceItemArr.length > 0) {
    hilog.info(0x0000, 'testTag', 'getResourceItemByCatalog get resource ', resourceItemArr[0]);
    return resourceItemArr[0];
  }  else if (spineList.length > 0) {
    hilog.info(0x0000, 'testTag', 'getResourceItemByCatalog get resource in resourceList', spineList[0]);
    return spineList[0];
  } else {
    hilog.info(0x0000, 'testTag', 'getResourceItemByCatalog get resource in escape');
    return {
      idRef: '',
      index: 0,
      href: '',
      properties: ''
    };
  }
}
```

---

## How Do I Build the Reader?

ReadPageComponent is the "soul" of the reader—page turns, progress, interaction, all on it.

First time I built a reader, I hid the loading before the page finished rendering—users got a black screen. Don't be me, add a loading state!

Here's how I do it:

```typescript
// Just copy this, but swap in your own paths and params
import { bookParser, ReadPageComponent, readerCore } from '@kit.ReaderKit';
import { BusinessError } from '@kit.BasicServicesKit';
import { display } from '@kit.ArkUI';
import { hilog } from '@kit.PerformanceAnalysisKit';
import { common } from '@kit.AbilityKit';

private readerComponentController: readerCore.ReaderComponentController = new readerCore.ReaderComponentController();
private readerSetting: readerCore.ReaderSetting = {
  fontName: 'System Font',
  fontPath: '',
  fontSize: 18,
  fontColor: '#000000',
  fontWeight: 400,
  lineHeight: 1.9,
  nightMode: false,
  themeColor: 'rgba(248, 249, 250, 1)',
  themeBgImg: '',
  flipMode: '0',
  scaledDensity: display.getDefaultDisplaySync().scaledDensity > 0 ? display.getDefaultDisplaySync().scaledDensity : 1,
  viewPortWidth: 370,
  viewPortHeight: 800,
};
private bookParserHandler: bookParser.BookParserHandler | null = null;
@State isLoading: boolean = true;

// Build the reading component
build() {
  Stack() {
    ReadPageComponent({
      controller: this.readerComponentController,
      readerCallback: (err: BusinessError, data: readerCore.ReaderComponentController) => {
        this.readerComponentController = data;
      }
    })

    Row() {
      Text('Loading...')
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
    .backgroundColor(Color.White)
    .visibility(this.isLoading ? Visibility.Visible : Visibility.None)
  }.width('100%').height('100%')
}

// Open the book at a specific spot
aboutToAppear(): void {
  let filePath: string = 'xxx/download/ebook/abc.epub';
  let spineIndex: number = 0;
  let domPos: string = '';
  this.registerListener();
  this.startPlay(filePath, spineIndex, domPos);
}

private registerListener(): void {
  this.readerComponentController.on('pageShow', (data: readerCore.PageDataInfo): void => {
    hilog.error(0x0000, 'testTag', 'pageshow: data is: ' + JSON.stringify(data));
    if (data.state === readerCore.PageState.PAGE_ON_SHOW) {
      this.isLoading = false;
    }
  });
}

private async startPlay(filePath: string, spineIndex: number, domPos: string) {
  try {
    let context = this.getUIContext().getHostContext() as common.UIAbilityContext;
    let initPromise = this.readerComponentController.init(context);
    let bookParserHandler = bookParser.getDefaultHandler(filePath);
    let result: [bookParser.BookParserHandler, void] = await Promise.all([bookParserHandler, initPromise]);
    this.bookParserHandler = result[0];
    this.readerComponentController.setPageConfig(this.readerSetting);
    this.readerComponentController.registerBookParser(this.bookParserHandler);
    this.readerComponentController.startPlay(spineIndex || 0, domPos);
  } catch (err) {
    hilog.error(0x0000, 'testTag', 'startPlay: err: ' + JSON.stringify(err));
  }
}

aboutToDisappear(): void {
  this.readerComponentController.off('pageShow');
  this.readerComponentController.releaseBook();
}
```

---

## Want Custom Fonts? Here's How

Sometimes the product says, "Can we use a different font?" and you have to support custom fonts. You can put font files in resources/rawfile/fonts or the sandbox path—just don't forget to register the resourceRequest callback, or the font won't load.

Here's how I do it:

```typescript
// Swap fonts—don't mess up the path, and don't forget the callback
import { fileIo as fs } from '@kit.CoreFileKit';
import { common } from '@kit.AbilityKit';
import { hilog } from '@kit.PerformanceAnalysisKit';

let filePath: string = 'fonts/SourceHanSerifCN-VF.ttf';
// let filePath: string = this.getUIContext().getHostContext()!.filesDir + 'fonts/SourceHanSerifCN-VF.ttf';

this.readerSetting.fontName = 'Source Han Serif';
this.readerSetting.fontPath = filePath;
this.readerComponentController.setPageConfig(this.readerSetting);

aboutToAppear(): void {
  this.readerComponentController.on('resourceRequest', this.resourceRequest);
}

aboutToDisappear(): void {
  this.readerComponentController.off('resourceRequest');
}

private isFont(filePath: string): boolean {
  let options = [".ttf", ".woff2", ".otf"];
  let path = filePath.toLowerCase();
  return options.some(ext => path.indexOf(ext) !== -1);
}

private resourceRequest: bookParser.CallbackRes<string, ArrayBuffer> = (filePath: string): ArrayBuffer => {
  if(filePath.length === 0){
    return new ArrayBuffer(0);
  }
  let resourcePath = filePath;
  if(this.isFont(filePath)){
    resourcePath = 'fonts/' + resourcePath;
  }
  try {
    let context = this.getUIContext().getHostContext() as common.UIAbilityContext;
    let value: Uint8Array = context.resourceManager.getRawFileContentSync(resourcePath);
    return value.buffer as ArrayBuffer;
  } catch (error) {
    return this.loadFileFromPath(resourcePath);
  }
}

private loadFileFromPath(filePath: string): ArrayBuffer {
  try {
    let stats = fs.statSync(filePath);
    let file = fs.openSync(filePath, fs.OpenMode.READ_ONLY);
    let buffer = new ArrayBuffer(stats.size);
    fs.readSync(file.fd, buffer);
    fs.closeSync(file);
    return buffer;
  } catch (err) {
    return new ArrayBuffer(0);
  }
}
```

---

## Custom Backgrounds? Go Wild

Want your reader to look unique? You can set custom background colors and images. If you use a light background, turn off night mode and make sure your font color matches—white on white is a classic rookie mistake.

Here's how I do it:

```typescript
// Change backgrounds—play with images and colors, but don't forget to match the font color
import { fileIo as fs } from '@kit.CoreFileKit';
import { common } from '@kit.AbilityKit';
import { hilog } from '@kit.PerformanceAnalysisKit';

this.readerSetting.themeColor = '#FFFFFF';
this.readerSetting.themeBgImg = '';
this.readerSetting.nightMode = false;
this.readerSetting.fontColor = '#000000';

this.readerSetting.themeBgImg = 'white_sky_first.jpg';
this.readerSetting.themeColor = '#FFFFFF';
this.readerSetting.nightMode = false;
this.readerSetting.fontColor = '#000000';
this.readerComponentController.setPageConfig(this.readerSetting);

aboutToAppear(): void {
  this.readerComponentController.on('resourceRequest', this.resourceRequest);
}

aboutToDisappear(): void {
  this.readerComponentController.off('resourceRequest');
}

private resourceRequest: bookParser.CallbackRes<string, ArrayBuffer> = (filePath: string): ArrayBuffer => {
  if(filePath.length === 0){
    return new ArrayBuffer(0);
  }
  try {
    let context = this.getUIContext().getHostContext() as common.UIAbilityContext;
    let value: Uint8Array = context.resourceManager.getRawFileContentSync(filePath);
    return value.buffer as ArrayBuffer;
  } catch (error) {
    return this.loadFileFromPath(filePath);
  }
}

private loadFileFromPath(filePath: string): ArrayBuffer {
  try {
    let stats = fs.statSync(filePath);
    let file = fs.openSync(filePath, fs.OpenMode.READ_ONLY);
    let buffer = new ArrayBuffer(stats.size);
    fs.readSync(file.fd, buffer);
    fs.closeSync(file);
    return buffer;
  } catch (err) {
    return new ArrayBuffer(0);
  }
}
```

---

## Don't Forget Dark/Light Mode

Everyone loves switching between dark and light mode these days. Reader Kit can adapt dynamically. Listen for colorMode changes, and when you switch themes, update both the font and background colors.

Here's how I do it:

```typescript
// Theme switching—don't forget to update both font and background colors
import { Configuration, UIAbility } from '@kit.AbilityKit';
import { ConfigurationConstant } from '@kit.AbilityKit';

export default class EntryAbility extends UIAbility {
  onConfigurationUpdate(newConfig: Configuration): void {
    AppStorage.setOrCreate('colorMode', newConfig.colorMode);
  }
}

@StorageLink('colorMode') @Watch('colorModeChange') colorMode: ConfigurationConstant.ColorMode =
  ConfigurationConstant.ColorMode.COLOR_MODE_NOT_SET;

colorModeChange() {
  if (this.colorMode === ConfigurationConstant.ColorMode.COLOR_MODE_DARK) {
    this.readerSetting.nightMode = true;
    this.readerSetting.fontColor = '#ffffff';
    this.readerSetting.themeColor = '#202224';
  } else {
    this.readerSetting.nightMode = false;
    this.readerSetting.fontColor = '#000000';
    this.readerSetting.themeColor = '#FFFFFF';
  }
  this.readerComponentController.setPageConfig(this.readerSetting);
}
```

---

## Pitfalls & Tips
- Only reads local books—don't expect to load from the cloud.
- DRM-protected books? Don't bother, they won't open.
- Only supports those formats—don't try pdf.
- Layout and interaction must use ReadPageComponent—don't try to reinvent the wheel.
- Only works on real devices—don't waste time on emulators.
- Mainland China only for now.
- If you hit a wall, don't panic—try the code, break things, and you'll figure it out. Got a cooler trick? Drop a comment!

---

## Official Docs & Community (Worth a Look)
- [Reader Kit Official Docs](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/reader-progress)
- [HarmonyOS Developer Community](https://developer.harmonyos.com/cn/community) 