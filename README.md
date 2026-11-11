# Personal Scoop Bucket

我自己常用 Windows 软件的 [Scoop](https://scoop.sh) bucket，由 GitHub Actions（Excavator）每日自动检查新版本并更新哈希。

## 使用方法

> 先把下面的 `<用户名>/<仓库名>` 替换为你在 GitHub 上的实际仓库地址。

```powershell
scoop bucket add my https://github.com/biglorange/scoop-buckets
scoop install my/coolapk-desktop
```

bucket 别名（`my`）可以随意改成自己喜欢的名字。

## 软件列表

| 应用 | 说明 | 主页 |
| --- | --- | --- |
| [coolapk-desktop](bucket/coolapk-desktop.json) | 酷安（Coolapk）第三方跨平台桌面客户端 | https://github.com/coolapk-desktop/coolapk-desktop |

## 自动更新机制

- [excavator.yml](.github/workflows/excavator.yml)：每天北京时间 02:00（UTC 18:00，可在 cron 里改）运行：
  1. 在 GitHub Actions 的 windows runner 上安装 Scoop；
  2. 调用 Scoop 自带的 `bin/checkver.ps1 -App '*' -Dir bucket -Update` 检查 `bucket/` 下所有 manifest 的新版本并原地更新（URL 用 `$version` 占位符，哈希由 Scoop 自动从 GitHub Release API 的 `digest` 字段提取，失败时回退为下载后本地计算 sha256）；
  3. 有变更则由 `github-actions[bot]` 直接提交并推送到 `main`。
- 也可在仓库 Actions 页面手动触发（`workflow_dispatch`）。
- [verify.yml](.github/workflows/verify.yml)：push / PR 时校验 `bucket/*.json` 的 JSON 语法和必备字段。
- 注意：GitHub 会在仓库 60 天无活动后自动暂停定时工作流，届时到 Actions 页面重新启用即可。

## 本地调试

```powershell
# 检查/更新本 bucket 的 manifest（假设 scoop 装在默认位置）
& "$env:USERPROFILE\scoop\apps\scoop\current\bin\checkver.ps1" -App '*' -Dir .\bucket -Update
```

## 添加新软件

复制 `bucket/coolapk-desktop.json` 作为模板，改掉 `version`/`url`/`hash`/`checkver`/`autoupdate` 后提交即可，Excavator 会接管后续更新。建议提交前跑一遍上面的本地检查命令确认 autoupdate 能正常产出新 manifest。
