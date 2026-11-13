<p align="center">
<img width="192" height="192" alt="ic_launcher" src="https://github.com/user-attachments/assets/f5eb03c2-3fe8-4436-b256-cfe41b28026b" />


</p>

<h1 align="center">ShareBox</h1>

<p align="center">

[English](README.md) | [简体中文](README.zh-CN.md)

**Android 手机与任意电脑、手机之间的 Wi-Fi 高速文件传输工具 —— 另一端什么都不用安装。**

ShareBox 把你的安卓手机变成一个小型文件服务器。电脑（Windows、Linux、macOS）或另一部手机只需要一个浏览器。它出自一位经常更换 Linux 发行版（Ubuntu、Debian 等）的用户，目的就是传文件时不必在电脑上装任何东西。

## 功能

- **对方零安装** —— 任何现代浏览器都能用：Windows、Ubuntu、Debian 及其他 Linux、macOS、另一部安卓手机或 iPhone。
- **浏览整个手机** —— 在浏览器里进入手机存储的每个文件夹，下载文件，删除文件或文件夹。
- **拖拽上传** —— 一次拖入多个文件，每个文件都有独立进度条。文件上传到你当前所在的文件夹。
- **命令行上传** —— 终端里一条 `curl` 命令即可，手机上会直接显示带当前访问码的完整命令。
- **速度快** —— 局域网内纯 HTTP，大文件流式传输。实际速度取决于 Wi-Fi（5GHz / Wi-Fi 6 下通常 30–100 MB/s）。
- **没有路由器也能用** —— 手机开热点，电脑连上去即可。
- **手机对手机** —— 另一部手机只需用浏览器打开地址。
- **息屏也不断** —— 以前台服务运行，并持有 Wi-Fi 锁和唤醒锁。
- **中英双语界面** —— 手机 App 跟随系统语言；网页跟随浏览器语言，并提供语言切换按钮。
- **深色主题**，手机 App 和网页都是。

## 安全
--- 防毒网站没有病毒,自己可以到这一个网址看一看：https://www.virustotal.com/gui/home/upload --- 
ShareBox 适合在可信的局域网内使用（家里的 Wi-Fi 或自己的热点）。

- **每次启动服务都会生成新的随机 6 位访问码**，旧码和已登录过的浏览器立即失效。
- 手机上有**“换访问码”按钮**，随时让所有登录失效。
- **防暴力破解** —— 同一设备连续输错 5 次，锁定 5 分钟。
- 登录成功后，访问码不会留在地址栏；登录状态使用 `HttpOnly` / `SameSite=Strict` 的 Cookie。
- 多人可以用同一个访问码同时登录，大家看到的文件和拥有的权限相同。

**局限：** 传输使用明文 HTTP，同一个 Wi-Fi 里有人抓包就能看到访问码和传输内容，请避免在公共 Wi-Fi 下使用。暂不支持 HTTPS。

## 使用方法

1. 安装 APK 并打开 ShareBox。首次启动请授予**“所有文件访问”**权限（Android 13+ 还需允许通知）。
2. 点**启动**。手机会显示类似 `192.168.0.9:8080` 的地址和 6 位访问码。
3. 确保电脑与手机在**同一个 Wi-Fi**（或电脑已连接手机热点）。
4. 在浏览器地址栏输入该地址（不必写 `http://`；如果浏览器自动改成 `https`，请手动加上 `http://`），然后输入访问码。
5. 浏览、上传、下载或删除。

命令行上传（手机上会直接显示这一行）：

```bash
curl -T app.apk "http://192.168.0.9:8080/put/app.apk?k=123456"
```

这种方式上传的文件默认放进 `Download/ShareBox`。想放到别的文件夹，在地址后加 `&p=某/文件夹`（相对于手机存储根目录的路径）。

## 编译

要求：Android 8.0+（minSdk 26）。使用 Kotlin 编写，唯一依赖是 [NanoHTTPD](https://github.com/NanoHttpd/nanohttpd)。

**Android Studio / Gradle**

```bash
gradle assembleDebug     # APK 位于 app/build/outputs/apk/debug/
```

**GitHub Actions** —— `.github/workflows/build.yml` 会在每次 push 时编译 APK，并作为 artifact 上传。

**纯手机流程（不用上传文件夹）** —— GitHub 手机网页版不能上传文件夹。把项目打成**一个 zip** 放在仓库根目录，再配上“Build from zip”工作流：它会自动解压、编译，并把 APK 发布到 **Releases** 页面，你可以直接下载到手机。

> 调试版 APK 在每次 CI 构建时签名都不同，Android 会拒绝覆盖安装。请先卸载旧版，或配置固定的签名密钥。

## 项目结构

```
app/src/main/
├── java/com/sharebox/
│   ├── MainActivity.kt    # 手机界面：启动/停止、地址、访问码
│   ├── ShareService.kt    # 前台服务、唤醒锁/Wi-Fi 锁、访问码
│   └── Server.kt          # HTTP 服务：登录、列表、下载、上传、删除
├── assets/index.html      # 提供给浏览器的网页（中/英）
└── res/                   # 字符串（en / zh）、桌面图标
```

## HTTP 接口

| 接口 | 作用 |
|---|---|
| `GET /` | 网页界面 |
| `GET /api/list?p=<目录>` | 列出文件夹 |
| `GET /f?p=<文件>` | 下载文件 |
| `POST /api/upload?p=<目录>` | 上传文件（multipart，字段名 `file`） |
| `PUT /put/<文件名>?p=<目录>` | 供 `curl -T` 使用的原始上传 |
| `POST /api/delete?p=<路径>` | 删除文件或文件夹 |

每个请求都需要访问码（`?k=` 或登录后的 Cookie）。路径始终限制在手机存储根目录内，`..` 越界会被拒绝。





<p align="center">
 <img width="192" height="192" alt="ic_launcher" src="https://github.com/user-attachments/assets/570943b5-a974-4f16-9926-fd59f6b81940" />


</p>

<h1 align="center">ShareBox</h1>

<p align="center">


[English](README.md) | [简体中文](README.zh-CN.md)

**Fast Wi-Fi file transfer between an Android phone and any computer or phone — with nothing to install on the other side.**

ShareBox turns your Android phone into a tiny file server. On the computer (Windows, Linux, macOS) or on another phone, you only need a web browser. It was built by a Linux user who constantly switches distributions (Ubuntu, Debian, …) and wants to move files without installing anything.

## Features

- **Zero install on the other side** — any modern browser works: Windows, Ubuntu, Debian, other Linux distributions, macOS, another Android phone or an iPhone.
- **Browse the whole phone** — navigate every folder of the phone's storage from the browser, download files, delete files or folders.
- **Drag-and-drop upload** — drop several files at once; each file has its own progress bar. Uploads go into the folder you are currently viewing.
- **Command-line upload** — one `curl` command from a terminal; the exact command (with the current access code) is shown on the phone.
- **Fast** — plain HTTP over your local Wi-Fi, large files are streamed. Real speed depends on your Wi-Fi (typically 30–100 MB/s on 5 GHz / Wi-Fi 6).
- **Works without a router** — turn on the phone's hotspot and connect the computer to it.
- **Phone to phone** — the second phone just opens the address in its browser.
- **Stays alive with the screen off** — runs as a foreground service with a Wi-Fi lock and a wake lock.
- **Bilingual UI (English / 简体中文)** — the phone app follows the system language; the web page follows the browser language and has a language switch button.
- **Dark theme** on both the phone app and the web page.

## Security

ShareBox is meant for a trusted local network (home Wi-Fi or your own hotspot).

- A random **6-digit access code is generated every time the service starts**. Old codes and already logged-in browsers stop working immediately.
- A **"Change access code" button** on the phone invalidates every session at any time.
- **Brute-force protection** — 5 wrong attempts from one device lock that device out for 5 minutes.
- After logging in, the code is removed from the address bar and the session cookie is `HttpOnly` / `SameSite=Strict`.
- Several people can be logged in at the same time with the same code; they all see the same files with the same rights.

**Limitations:** traffic is plain HTTP, so someone sniffing the same Wi-Fi could read the code and the transferred data. Avoid public Wi-Fi. HTTPS is not supported yet.

## Usage

1. Install the APK and open ShareBox. On first launch grant the **All files access** permission (and notifications on Android 13+).
2. Tap **Start**. The phone shows an address such as `192.168.0.9:8080` and a 6-digit access code.
3. Make sure the computer is on the **same Wi-Fi** (or connected to the phone's hotspot).
4. Type the address into the browser (no need to write `http://`; if a browser forces `https`, type `http://` explicitly) and enter the access code.
5. Browse, upload, download or delete.

Command-line upload (the phone displays this line for you):

```bash
curl -T app.apk "http://192.168.0.9:8080/put/app.apk?k=123456"
```

Files uploaded this way go to `Download/ShareBox` by default. Append `&p=Some/Folder` to choose another folder (path relative to the phone's storage root).

## Build

Requirements: Android 8.0+ (minSdk 26). Written in Kotlin with [NanoHTTPD](https://github.com/NanoHttpd/nanohttpd) as the only dependency.

**Android Studio / Gradle**

```bash
gradle assembleDebug     # APK: app/build/outputs/apk/debug/
```

**GitHub Actions** — `.github/workflows/build.yml` builds the APK on every push and uploads it as an artifact.

**Phone-only workflow (no folder upload needed)** — GitHub's mobile web UI cannot upload folders. Put a single `.zip` of the project in the repository root together with the "Build from zip" workflow: the workflow unzips it, builds the APK and publishes it on the **Releases** page, where you can download it straight to the phone.

> Debug builds are signed with a different key on every CI run, so Android refuses to update over a previous build. Uninstall the old version first, or add a fixed keystore.

## Project structure

```
app/src/main/
├── java/com/sharebox/
│   ├── MainActivity.kt    # phone UI: start/stop, address, access code
│   ├── ShareService.kt    # foreground service, wake/Wi-Fi locks, access code
│   └── Server.kt          # HTTP server: login, list, download, upload, delete
├── assets/index.html      # the web page served to the browser (EN/中文)
└── res/                   # strings (en / zh), launcher icons
```

## HTTP endpoints

| Endpoint | Purpose |
|---|---|
| `GET /` | web UI |
| `GET /api/list?p=<dir>` | list a folder |
| `GET /f?p=<file>` | download a file |
| `POST /api/upload?p=<dir>` | upload a file (multipart, field `file`) |
| `PUT /put/<name>?p=<dir>` | raw upload for `curl -T` |
| `POST /api/delete?p=<path>` | delete a file or folder |

Every request needs the access code (`?k=` or the login cookie). Paths are always resolved inside the phone's storage root; `..` escapes are rejected.

