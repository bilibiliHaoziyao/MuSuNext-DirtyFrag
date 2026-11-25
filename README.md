# DirtyFrag — MuSu Next locked fork

本目录是 DirtyFrag 引擎的独立 fork 源码树（GPL-2.0-only）。它不参与 ksud / manager 的任何编译单元，仅作为独立 APK 引擎与独立进程运行；MuSu Next 只通过进程边界调用它。见 `docs/specs/dirtyfrag.md` §2/§3/§4。

## ① 上游信息与 fork 锚点

- 上游 URL：https://github.com/mitschud/DirtyFrag
- 上游 tag：`v1.18`（annotated tag 对象 `7b86de6bf2ede38c4bb3361e8581739acebd40a7`）
- 上游 commit（固定构建点）：`3bad3cd6f7a9ce0a5536d900af264be2645d1512`
- 上游 commit 日期：`2026-11-17`
- fork 仓库：https://github.com/bilibiliHaoziyao/MuSuNext-DirtyFrag
- fork commit：以 fork 仓库 HEAD 为准（提交哈希不能写进其自身提交内的文件；`SOURCE.txt` 只固定上游锚点）
- fork 日期：`2026-11-25`
- 上游源码快照：`dirtyfrag/upstream/`（逐字取自上述 commit 的 GitHub API tarball，未改动）

## ② 修改清单（对应 `patches/`）

| patch | 改动 | 文件 |
|---|---|---|
| `0001-add-musu-manager.patch` | SU Manager 白名单加入 `org.musu.next`；`libksud.so` 来源指向该管理器；运行期拒绝非白名单选择 | `app/src/main/java/df/root/ExploitRunner.java`, `MainActivity.java` |
| `0002-locked-mode.patch` | 新增锁定模式：intent extra `org.musu.next.dirtyfrag.LOCKED_MANAGER=org.musu.next` 命中则固定管理器、跳过选择器 UI；extra 值不符时拒绝运行 | `app/src/main/java/df/root/MainActivity.java` |
| `0003-no-launcher.patch` | 移除 `android.intent.category.LAUNCHER`（无独立桌面图标），保留 MAIN action + `exported=true`，由 MuSu Next 显式组件名拉起 | `app/src/main/AndroidManifest.xml` |

三个 patch 按序可应用到固定 commit：
```bash
cd dirtyfrag/upstream
for p in ../patches/000{1,2,3}-*.patch; do git apply --check "$p" && git apply "$p"; done
```

## ③ 许可证

- 许可证：**GPL-2.0-only**（不是 GPL-2.0-or-later，也不是 GPL-3.0）。
- 原文：`dirtyfrag/LICENSE`（与上游 `upstream/LICENSE` 逐字一致，sha256 `81d2e2140695d3a3c76c1fa48b8fc34abfee067f6197812c3e842612be5c3ff6`）。
- 修改声明与版权：`dirtyfrag/NOTICE`。
- 来源锚：`dirtyfrag/SOURCE.txt`。
- 由于本 fork 有自己的修改，依 GPL-2.0 §3 随分发提供修改源码：`dirtyfrag/upstream/` + `dirtyfrag/patches/`。

## ④ 兼容性声明

上游原文（`upstream/README.md` License 段）：
> GPL-2.0-only cannot absorb GPL-3.0-only code. A GPL-3 reference ... may be read for behaviour, record formats and API naming, and none of its code may be copied, translated or linked into this tree.

我们的结论：
> DirtyFrag is GPL-2.0-only. MuSu Next userspace is GPL-3.0-or-later; they are distributed as separate programs (mere aggregation) with no code copied, translated or linked across the boundary. See `docs/specs/dirtyfrag.md` §2.

成立条件：独立 APK / 独立进程 / 独立 UID；仅通过 Intent、drop-box 文件与单 JSON 交互；无 dlopen / FFI / 共享地址空间；编排层零移植漏洞链。

## ⑤ 构建命令与 SHA256 清单

### 构建命令（在已有 Android SDK / NDK 的 CI runner 上）

```bash
# 前置：Android SDK platform 37 + build-tools 37 + NDK 29 + CMake；Java 21
cd <fork checkout>
printf 'sdk.dir=/opt/android-sdk\n' > local.properties     # 按 runner 实际路径
# 签名：app/build.gradle.kts 使用 app/keystore.jks（gitignored）。
# CI 用 secrets.KEYSTORE（base64 of JKS）解码到 app/keystore.jks：
#   echo "$KEYSTORE" | base64 --decode > app/keystore.jks
# 本地自建（仅用于验证，勿用于发布）：
keytool -genkeypair -keystore app/keystore.jks -storetype JKS \
  -storepass dirtyfrag -keypass dirtyfrag -alias dirtyfrag \
  -keyalg RSA -keysize 2048 -validity 10000 -dname "CN=MuSu Next DirtyFrag Fork"
# 三个 patch 必须先应用（见 §②），然后：
JAVA_HOME=<jdk21> ANDROID_HOME=<sdk> ./gradlew :app:assembleRelease --no-daemon
# 产物：app/build/outputs/apk/release/dirtyfrag.apk
```

注意：发布用签名证书必须由 MuSu Next 自己持有并固定指纹（`SHA256SUMS` 需记录 APK 的 sha256）；不要用上游的发布签名。

### SHA256 清单

8 个 ko 直接取自上游固定 commit 的 `app/src/main/jni/ko/`，未重编译、未改名、未替换：

```
c4721299599b39552d3241624ce19e657f0f2e98654392974d47f027d3a3a39f  dfroot-android12-5.10.ko
edfb34ffb2addc2a85a8a3d6d164ed279636afc72526185b01939e1a5f703a19  dfroot-android13-5.10.ko
027a2c6e68b8fae38ee2f6f7ed0cbccb0838d78187e086e1c62af0c4e1a0be79  dfroot-android13-5.15.ko
2e70c398fddc69f511f1772db78c7e3508c2b4d87c5905bd8967eb35fb1a62ac  dfroot-android14-5.15.ko
5af4e7c94fee4e8e30fc5519d8382573b34c7eed5fdf5ffd13d4e2b786aa5752  dfroot-android14-6.1.ko
80ea9cc65be4e68d9c9a7c3ef8ae18119089d73ba233ed1003a61cfeec32ab3c  dfroot-android15-6.6.ko
a51e2af7595fb2cb35e2e2741f0c351ba68c3380ae7d06586861d49ccf965986  dfroot-android16-6.12.ko
e55766318e9821ad567686c931d396e22acfc12ef23ee31bf51e2692ca99948c  dfroot-android17-6.18.ko
```

APK（锁定版，应用三个 patch 后本机真实构建，非占位）：
```
2ba8f9be4ccd6652d03ef0420be52695e33fc196d89b9cd367899f77c679c42e  dirtyfrag-locked.apk
```
- 构建结果：`BUILD SUCCESSFUL in 2m 41s`（46 tasks），产物 `dirtyfrag.apk` 310661 bytes，ZIP 完整性 `No errors detected`，`apksigner` v2 验签通过。
- 签名证书 DN：`CN=MuSu Next DirtyFrag Fork`，证书 SHA-256 指纹 `338d89472d8038836b6623298e92e6eb9dd736f159a70eb2812a0f520acb1253`（本地验证签名；发布须换 MuSu Next 正式签名并固定其指纹）。
- 已从 dex 中确认补丁字符串 `org.musu.next.dirtyfrag.LOCKED_MANAGER`、`unsupported SU manager:`；`aapt2 dump badging` 无 `launchable-activity`（LAUNCHER 已移除）。

所有 ko 均为 AArch64 ELF（`\x7fELF` 魔数，`e_machine=183`），大小 10–15 KB，来自真实内核模块源码，不是占位文件。真实清单见 `userspace/ksud/bin/dirtyfrag/arm64/SHA256SUMS`。

## ⑥ 锁定模式复现

MuSu Next 以显式组件名启动引擎，并携带 intent extra：

- extra 名：`org.musu.next.dirtyfrag.LOCKED_MANAGER`
- 期望值：`org.musu.next`（MuSu Next 包名；PR 构建为 `org.musu.next.pr`，此时 extra 值仍为 `org.musu.next`）
- 行为：命中后固定 SU manager 为 `org.musu.next`，跳过 SU-manager 选择器；extra 值不符则拒绝运行并记错误日志。
- 组件：`df.root/.MainActivity`（已移除 LAUNCHER category，无桌面图标；`exported=true` 允许显式启动）。

示例（adb，需已安装锁定版引擎）：
```bash
adb shell am start -n df.root/.MainActivity \
  --es org.musu.next.dirtyfrag.LOCKED_MANAGER org.musu.next
```
