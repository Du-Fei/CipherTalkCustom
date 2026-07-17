# 用户数据存储与清除

本文说明 CipherTalk 在 Windows 上产生的数据存储在哪里，以及如何按需清除数据。

> 清除前必须完全退出 CipherTalk，包括系统托盘中的进程。正在运行时删除数据库可能导致文件被占用或再次产生不完整数据。

## 数据存储位置

CipherTalk 的用户数据分为应用配置和业务缓存两类。

### 应用配置目录

Electron 用户数据默认位于：

```text
%APPDATA%\CipherTalk
```

在 PowerShell 中可以通过以下命令查看实际路径：

```powershell
Join-Path $env:APPDATA 'CipherTalk'
```

该目录通常包含：

- `ciphertalk-config.db`：账号、微信数据路径、缓存路径、解密参数、HTTP API 和界面设置。
- `Cache`、`Code Cache`、`GPUCache`：Electron 页面缓存。
- `Local Storage`、`Session Storage`：页面状态。
- `stt-cache.db`、`whisper-models`：语音识别缓存和模型。

### 解密数据库和媒体缓存

实际位置由设置中的“缓存路径”决定。常见默认位置为：

```text
%USERPROFILE%\Documents\CipherTalkData
```

如果安装版位于非系统盘，默认缓存也可能位于安装目录下的 `CipherTalkData`。用户主动选择过缓存目录时，以设置中显示的路径为准。

该目录通常包含：

- 以微信账号命名的目录：解密后的 SQLite 数据库。
- `Images`、`Emojis`：解密媒体和表情缓存。
- `logs`：运行日志。
- `chat_search_index.db`：全文搜索索引。
- `stt-cache.db`、语音模型目录：语音识别数据。

## 仅重建损坏的解密数据库

当日志出现以下错误时，可以只清除当前账号的解密数据库：

```text
SQLITE_CORRUPT: database disk image is malformed
```

操作步骤：

1. 完全退出 CipherTalk。
2. 打开设置中配置的缓存目录。
3. 删除对应微信账号的子目录。不要删除微信原始数据目录。
4. 重新启动 CipherTalk，执行完整解密或同步。
5. 保持“跳过数据库完整性检查”关闭。

例如，缓存目录为 `D:\CipherTalkDB`、账号目录为 `<wxid>` 时：

```powershell
Remove-Item -LiteralPath 'D:\CipherTalkDB\<wxid>' -Recurse -Force
```

请先将示例中的路径和 `<wxid>` 替换为界面中显示的实际值。此操作保留应用设置、API Token 和其他账号的数据。

## 仅清除界面缓存

如果只是界面显示异常，可保留配置数据库，仅删除 Electron 缓存：

```powershell
$userData = Join-Path $env:APPDATA 'CipherTalk'
@('Cache', 'Code Cache', 'GPUCache', 'DawnGraphiteCache', 'DawnWebGPUCache') |
  ForEach-Object {
    $target = Join-Path $userData $_
    if (Test-Path -LiteralPath $target) {
      Remove-Item -LiteralPath $target -Recurse -Force
    }
  }
```

## 完全重置 CipherTalk

完全重置会删除所有 CipherTalk 配置、解密数据库、媒体缓存、搜索索引、日志和下载的语音模型。

1. 完全退出 CipherTalk。
2. 确认设置中的实际缓存路径。
3. 删除应用配置目录。
4. 删除实际缓存目录。
5. 重新启动 CipherTalk 并重新配置账号。

```powershell
# 删除应用配置和 Electron 缓存
$userData = Join-Path $env:APPDATA 'CipherTalk'
if (Test-Path -LiteralPath $userData) {
  Remove-Item -LiteralPath $userData -Recurse -Force
}

# 将此处替换为设置中显示的实际缓存路径
$cachePath = 'D:\CipherTalkDB'
if (Test-Path -LiteralPath $cachePath) {
  Remove-Item -LiteralPath $cachePath -Recurse -Force
}
```

## 安全注意事项

- 不要删除设置中“微信数据路径”指向的目录。它是微信原始数据，不是 CipherTalk 缓存。
- 缓存路径可能由用户修改，不应假定所有设备都使用同一个盘符或目录。
- 执行 `Remove-Item -Recurse -Force` 前，应先输出并核对目标路径：

```powershell
$userData
$cachePath
```

- 如需保留设置，可先备份 `%APPDATA%\CipherTalk\ciphertalk-config.db`。
- 导出到用户指定目录的 JSON、HTML、图片等文件不属于应用缓存，不会随上述操作自动删除。
