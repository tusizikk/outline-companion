# 用 GitHub Actions 构建 APK（上传 zip 版）

你只需要上传 **一个 zip 文件**，再在 GitHub 网页里新建一个 workflow 文件，Actions 就会在云端编译 APK。

## 1. 新建仓库

GitHub → New repository → 名称例如 `outline-companion` → Create repository。

不需要勾选 README / .gitignore。

## 2. 上传 zip

仓库首页 → **Add file → Upload files** → 选择本目录外的 `android-outline-app.zip` → Commit changes。

确认仓库根目录能看到 `android-outline-app.zip`。

## 3. 新建 workflow

仓库首页 → **Add file → Create new file**，文件名填：

```
.github/workflows/build-from-zip.yml
```

把下面整段粘进去，Commit：

```yaml
name: Build APK from zip

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Unzip project
        run: unzip -q android-outline-app.zip

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Set up Android SDK
        uses: android-actions/setup-android@v3

      - name: Accept Android SDK licenses
        run: yes | sdkmanager --licenses || true

      - name: Install Android platform and build tools
        run: sdkmanager "platforms;android-34" "build-tools;34.0.0"

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.5'

      - name: Build debug APK
        run: gradle assembleDebug --no-daemon

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: outline-companion-debug
          path: app/build/outputs/apk/debug/app-debug.apk
```

## 4. 运行

仓库 **Actions** → 左侧 **Build APK from zip** → **Run workflow** → Run。

等 3–6 分钟，进入这次 run，页面底部 **Artifacts** 下载 `outline-companion-debug`。

解压后得到 `app-debug.apk`，传到手机安装。

## 5. 手机设置

1. AnkiDroid → 设置 → 高级 → 启用 **AnkiDroid API**；
2. 允许安装未知来源 APK；
3. 打开 App，允许 `READ_WRITE_DATABASE` 权限；
4. 点「读取大纲卡笔记」。

## 如果构建失败

把 Actions 里失败步骤的红色日志截图/文字发我，我改代码或 workflow。

## 备注

- 这个 POC 是**只读**的，不会改任何 note；
- 如果上传的是本目录的源码而不是 zip，仓库自带的 `.github/workflows/android.yml` 会在 push 时自动构建，不需要第 3 步；
- 私有仓库也能用 Actions，但会消耗账户的 Actions 分钟数。
