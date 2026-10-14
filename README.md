# 图片分类（PhotoClassifier）v2.0

一个把手机里成百上千张图片**逐张分到不同文件夹**的安卓小工具。

| 项目 | 说明 |
| --- | --- |
| 包名 | `com.photoclassify.app` |
| 版本 | 2.0（versionCode 2） |
| 最低系统 | **Android 7.0（API 24）** |
| 编译 / 目标 | compileSdk 37 / targetSdk 36 |
| 技术栈 | Java + AndroidX（AppCompat / Material 3 / RecyclerView / ViewPager2） |
| 产物 | `app/build/outputs/apk/debug/app-debug.apk` |
| 作者链接 | https://github.com/wzh086/ai-AI-Android-Image-Classification-Software/upload/main |

---

## 一、v2.0 更新了什么

### 修复的 Bug

| 编号 | 现象 | 原因 | 修复位置 |
| --- | --- | --- | --- |
| 1 | 分区全部解散后，左上角"未分类 / 剩余"数字没有恢复原样 | 解散后重建队列时仍沿用旧下标，出现"未分类 100 / 剩余 70"这类数字 | `ClassifySession.setQueue()` 重建时下标归零；`ClassifyActivity` 的解散 / 合并 / 排序改为 `reloadQueue(false)` |
| 2 | 清空记录再重新扫描，进入第二屏看到的不是 1 号图 | 同上：进程级队列单例里的旧下标被保留 | 同上（`setQueue` 归零）+ 清空记录时额外调用 `ClassifySession.clear()` |

> 实测结果：清空记录 → 重新扫描 → 第二屏显示"未分类 10 / 剩余 10 · 编号 #1"；
> 解散分区后同样回到"未分类 10 / 剩余 10 · 编号 #1"。

### 新增的功能

1. **异形屏适配**：刘海 / 挖孔 / 水滴屏全部适配，同时兼容 Android 15 起的"边到边"强制显示
   （`util/EdgeToEdge.java` + `values-v28/themes.xml` 的 `shortEdges`）。
   注意点：顶栏改成了 `wrap_content + minHeight`，这样加上状态栏内边距后标题不会被压扁。
2. **图片预加载**：自动提前解好后面两张，翻页几乎瞬间出图（`ImageLoader.preload()`）。
3. **分区相册**：在第二屏**长按分区键**，一次看完该分区所有照片，点任意一张可放大
   （`SectionBrowserActivity`）。大图页里还能把单张"移出分区"，退回未分类库。
4. **新手引导**：首次安装自动出现，可跳过；之后从第一屏菜单栏随时再看（`GuideActivity`）。
5. **使用说明书**：所有菜单栏里都有入口，全文在 `strings.xml` 的 `manual_body`（`ManualActivity`）。
6. **图片区背景透明**：直接露出应用背景（纯色 / 插画 / 自定义图片），只留一圈细边框。
7. **主题系统**（`theme/` 包 + `ThemeActivity`）：
   - 五套内置主题：**经典（黄绿）**、**纯白**、**纯黑（夜间）**、**鲸鱼娘·卡通版**、**鲸鱼娘·成熟版**；
   - 自定义颜色：背景、顶栏、菜单、分区栏、分区键、新建分区键、按键文字、按钮、卡片、边框；
   - 自定义背景图片：从相册选图，自动居中裁切 + 加遮罩保证文字可读；
   - 底部实时预览。
   - 按钮上色用 `setBackgroundTintList()`（因为 `<Button>` 会被 Material 主题换成
     MaterialButton，它自己管理背景，直接 `setBackground()` 会被忽略）。
8. **关于页**：显示版本号、作者链接（可点击），并注明"其他版本见链接"。
9. **逐行中文注释**：每个 Java / XML 文件都有文件头说明 + 逐行注释，讲清"这行干什么、怎么改"。

---

## 二、用 Android Studio 打开与运行

1. 打开 Android Studio → **Open** → 选择本文件夹（名字是 `1`）。
2. 首次打开会自动同步 Gradle（右上角出现 **Sync Now** 就点一下）。
   本工程已自带：
   - Gradle Wrapper（Gradle 9.6.0）
   - `local.properties`（指向本机 SDK：`D:\pss\ad\Sdk`）
   - `keystore/debug.keystore`（自带 debug 签名，换电脑也能直接出可安装包）
3. 点绿色三角 **Run** 直接装到手机 / 模拟器；或点 **Make Project** 只编译。

### 命令行构建

```bat
gradlew.bat assembleDebug
```

APK 在 `app\build\outputs\apk\debug\app-debug.apk`。
安装到手机：`adb install -r app\build\outputs\apk\debug\app-debug.apk`
（或直接把 APK 拷进手机点击安装，需要允许"安装未知来源应用"）。

### 怎么预览 `app/res/layout` 下的界面

Android Studio 打开任意布局文件后，右上角切到 **Design** 或 **Split** 就能看到实时预览。
本工程已经为预览做好了准备：

- 所有布局都带 `tools:` 预览属性（`tools:text`、`tools:src`、`tools:listitem`、`tools:itemCount`），
  所以 RecyclerView、文字、图片在预览里都有内容，不是空白；
- `item_section.xml` 用了 `SectionKey` 样式，预览里就是真实的分区键外观；
- 想预览大图页的深色效果，把预览面板顶部的 **Theme** 从 `Theme.PhotoClassifier`
  换成 `Theme.PhotoClassifier.Viewer`；
- 注意：**具体颜色是运行时由主题引擎刷上去的**，所以预览里看到的是"经典主题"的静态样式。
  想看其他主题的真实效果，请运行到手机 / 模拟器上切换主题。

### 每个布局文件对应哪个界面

| 布局文件 | 对应界面 |
| --- | --- |
| `activity_main.xml` | 第一屏：扫描图片 |
| `activity_classify.xml` | 第二屏：逐张分类（图片区 + 分区栏） |
| `activity_viewer.xml` | 大图页：缩放 / 翻页 / 就地分类 |
| `activity_section_browser.xml` | 分区相册（长按分区键进入） |
| `activity_guide.xml` | 新手引导 |
| `activity_manual.xml` | 使用说明书 |
| `activity_theme.xml` | 主题设置 |
| `item_section.xml` | 分区栏里的一个键 |
| `item_photo_grid.xml` | 分区相册里的一个格子 |
| `item_guide_page.xml` | 引导里的一页 |
| `item_color_row.xml` | 主题设置里的一行颜色 |
| `dialog_about.xml` / `dialog_new_section.xml` / `dialog_merge.xml` | 三个对话框 |

---

## 三、目录与文件（每一处都有中文注释）

```
1/
├── settings.gradle            工程名与子模块
├── build.gradle               插件版本（AGP 9.4.0）
├── gradle.properties          JVM 内存、AndroidX 开关
├── local.properties           Android SDK 路径（换电脑要改）
├── gradlew / gradlew.bat      命令行构建脚本
├── keystore/debug.keystore    自带 debug 签名
└── app/
    ├── build.gradle           应用配置（包名、版本、依赖）
    ├── proguard-rules.pro     混淆规则（当前未开启混淆）
    └── src/main/
        ├── AndroidManifest.xml        权限与界面登记
        ├── java/com/photoclassify/app/
        │   ├── App.java               应用入口（预热主题）
        │   ├── MainActivity.java      第一屏
        │   ├── ClassifyActivity.java  第二屏（核心）
        │   ├── ViewerActivity.java    大图页（两种模式）
        │   ├── SectionBrowserActivity.java  分区相册
        │   ├── GuideActivity.java     新手引导
        │   ├── ManualActivity.java    使用说明书
        │   ├── ThemeActivity.java     主题设置
        │   ├── data/                  数据层
        │   │   ├── Photo.java             一张图片的记录
        │   │   ├── PhotoDb.java           编号总库（JSON 持久化）
        │   │   ├── Scanner.java           按扩展名扫描 + 合并编号
        │   │   ├── SectionStore.java      photo/ 分区文件夹与文件搬运
        │   │   ├── ClassifySession.java   分类队列（两个 Bug 的修复点）
        │   │   ├── SectionBrowseSession.java 分区浏览队列
        │   │   ├── ImageLoader.java       解码 / 缓存 / 预加载
        │   │   └── Perms.java             权限与图片格式判断
        │   ├── theme/                 主题引擎
        │   │   ├── AppTheme.java          一套主题的数据结构
        │   │   └── ThemeManager.java      四套内置主题 + 换肤 + 自定义
        │   ├── ui/                    自定义控件
        │   │   ├── SectionAdapter.java    分区键适配器
        │   │   ├── SpacingDecoration.java 等距间距
        │   │   ├── ZoomImageView.java     可缩放大图控件
        │   │   ├── PhotoGridAdapter.java  分区相册网格
        │   │   └── GuidePagerAdapter.java 引导翻页
        │   └── util/
        │       ├── EdgeToEdge.java    异形屏 / 边到边适配
        │       └── Ui.java            对话框、版本号、链接等小工具
        └── res/
            ├── layout/    14 个布局（全部可预览）
            ├── menu/      5 个菜单
            ├── values/    strings / colors / dimens / themes / arrays / ids
            ├── values-v28/themes.xml   刘海屏属性（API 28+）
            ├── values-v29/themes.xml   关闭强制反色（API 29+）
            ├── drawable/  图标与背景
            ├── drawable-nodpi/whale_bg.png  鲸鱼娘主题的原创插画
            ├── mipmap-*/  应用图标（含自适应图标与单色图层）
            └── xml/file_paths.xml       FileProvider 白名单
```

---

## 四、编号规则（核心机制）

| 情况 | 处理方式 |
| --- | --- |
| 文件还在记录的路径上 | 直接沿用原编号，分类状态不变 |
| 文件被别的应用挪走了 | 用"文件名 + 大小"把旧记录认回来，编号不变 |
| 记录里有、磁盘上没有 | 标记为"丢失"，不进分类队列，文件回来还能复活 |
| 磁盘上有、记录里没有 | 当作新图片，补一个新编号 |

编号库位置：`/data/data/com.photoclassify.app/files/photo_index.json`（卸载应用即清除，不动你的图片）。

分区目录：`存储根目录/photo/分区名/`（`photo` 只在第一次运行时创建）。

---

## 五、权限说明

| 系统版本 | 需要的权限 |
| --- | --- |
| Android 7 ~ 9 | `READ_EXTERNAL_STORAGE` + `WRITE_EXTERNAL_STORAGE` |
| Android 10 | 同上（清单里开了 `requestLegacyExternalStorage` 走旧模式） |
| Android 11 ~ 12 | `READ_EXTERNAL_STORAGE` + **所有文件访问权限** |
| Android 13+ | `READ_MEDIA_IMAGES` + **所有文件访问权限** |

"所有文件访问权限"是必须的：要在存储根目录建 `photo` 文件夹、并把图片在目录之间搬来搬去，
只能走直接文件路径。第一屏点扫描时会弹窗引导去系统设置里开启；
不开启也能选"仅用媒体库扫描"，但分类移动可能失败。

---

## 六、关于两个鲸鱼娘主题

一共两套，**用的是同一套角色配色，区别在角色立绘与整体深浅**：

| 主题 | 角色立绘 | 风格 |
| --- | --- | --- |
| 鲸鱼娘 · 卡通版 | Q 版正面全身 | 明亮活泼：中蓝顶栏 `#4969B1` + 浅蓝分区栏 |
| 鲸鱼娘 · 成熟版 | 成熟版正面全身 | 沉稳优雅：深藏青顶栏 `#232C56` + 冷灰蓝分区栏，夜里更耐看 |

- 配色从角色设定图里采样得到：深藏青 `#29366A`（裙）+ 中蓝 `#4969B1`（头发/领结）
  + 米白 `#F1EEF4`（围裙花边）+ 金色 `#C9A227`（刺绣点缀）。
- 背景图 `whale_chibi_bg.jpg` / `whale_mature_bg.jpg` 由 `tools/gen_whale_from_ref.py` 生成：
  从设定图里**自动抠掉纯白背景**（人物内部的白色围裙、花边不会被误删），
  取出正面全身那一格，再按手机竖屏重新排版（渐变底 + 气泡 + 金色小星星 + 文字可读性柔化层）。
  角色像素完全来自你提供的设定图，脚本只做抠图与排版，不会重绘角色。
- 想换角色立绘或调整大小：改脚本 `main()` 里的 `region`（裁剪范围）与
  `figure_scale`（角色占屏幕高度比例）、`figure_y`（顶部留白比例），重新运行即可。
- 想重新生成，需要把两张设定图放在 `C:\Users\Admin\Desktop\deepseek娘\` 下，
  或修改脚本顶部的 `SRC_CHIBI` / `SRC_MATURE` 路径。

> 提醒：这两张设定图来自网络检索，如果以后要**公开发布**这个应用，
> 建议先确认图片的授权与署名要求，或者在仓库里注明来源。

---

## 七、隐私

所有处理都在手机本地完成，**不联网、不上传任何图片**；
申请的权限只用于读写你自己的存储。
