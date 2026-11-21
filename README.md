# DualLink 发布仓库

家用智能数字电子钟（STC8H8K64U + DX-WF30 + Android App）的版本清单与 OTA 分发仓库。

## 目录

- `version.json` —— App 和固件共用的版本清单，App 启动时读取它并比对版本号
- `docs/OTA协议.md` —— 设备端固件升级协议（`OTA:BEGIN / DATA / END`）

## App 读取 version.json 的三种地址

| 用途 | 地址 |
|---|---|
| 原始文件（GitHub CDN） | `https://raw.githubusercontent.com/zi-tian/DualLink/main/version.json` |
| jsDelivr 镜像（国内更稳） | `https://cdn.jsdelivr.net/gh/zi-tian/DualLink@main/version.json` |
| 局域网兜底 | `http://<PC 的 IP>:8000/version.json` |

二进制文件（APK、hex）放在 Release 资产里，不走 raw，地址形如：

```
https://github.com/zi-tian/DualLink/releases/download/v1.0/DualLinkCtrl.apk
```

## 判断规则

- 只看 `versionCode`（整数），**不要**用 `versionName` 字符串比大小
- `json.versionCode > 本地 versionCode` 才提示更新
- `fw.version` 和设备回 `VER?` 的值比对，低于才提示推固件
- `force: true` 时对话框不可取消

## 发版步骤

1. 改 `App\Wf30Ctrl\build.ps1` 里 `--version-code` 和 `--version-name`，重新打包
2. 在 GitHub 页面 Releases → Draft a new release：tag 用 `v<版本号>`，把 `DualLinkCtrl.apk`（需要就再加 `DualLink_vX.Y.hex`）拖进去发布
3. 把 `version.json` 里的 `versionCode / versionName / apk 地址 / size / md5` 同步更新并提交
4. 手机上的 App 下次启动就会读到新版本

## 校验值怎么取

Windows PowerShell：

```powershell
(Get-Item .\DualLinkCtrl.apk).Length
(Get-FileHash .\DualLinkCtrl.apk -Algorithm MD5).Hash.ToLower()
```
