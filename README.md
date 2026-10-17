# MMUUAI CLI

独立的 MMUUAI 命令行客户端，可供终端、脚本和 Agent 使用。与插件、MCP 接入共享现有平台模型、工作流、技能、提示词、会员权益和任务账本。

需要 Node.js 24.16 或更高版本，以及 Python 3（复用平台插件的上传、保存与下载命名脚本）。安装包已包含 MCP 官方 SDK，不需要 MMUUAI 业务源码，也不安装额外运行依赖。

上传与 MCP 连接使用同一套显式代理环境设置（`HTTP_PROXY`、`HTTPS_PROXY`、`NO_PROXY`），不另外启用 Windows 注册表代理，不修改系统全局设置。网络失败会附安全的 `transportReason`；已有 HTTP 响应时可附 `httpStatus` 和 `requestId`，均不包含一次性上传地址或凭据。未知上传不会自动重试。

## 安装

解压官方 ZIP 到持久目录，在该目录执行 `npm install --global .`，再执行 `mmuuai --version`。请保留解压目录；本地 npm 包可能通过链接安装。若系统 npm 全局目录无写入权限，使用当前用户的 npm prefix，不需要修改系统安全设置。

安装包的 `skills/mmuuai-cli/` 是配套使用 Skill；Agent 可按其实际支持的 Skill 安装方式加载。已有 MMUUAI 插件时复用现有 Skill，不重复建立同一 MCP 连接。

## 登录

运行 `mmuuai login`，在浏览器中的 MMUUAI 官方授权页登录、选择工作区并确认。也可用 `mmuuai login --no-browser` 取得授权链接，由用户在浏览器完成。密码、Cookie、授权码和 Token 不需要发送给 Agent。`mmuuai logout` 撤销本工具连接并清理本机凭据。

Windows 凭据使用当前 Windows 账号的 DPAPI 保护；其他系统保存在当前用户的私有配置目录，目录权限 0700、文件权限 0600。CLI 不读取 Codex 或其他软件的凭据。受控脚本也可通过 `MMUUAI_API_KEY` 环境变量使用平台 API Key，密钥不作为命令行参数。

## 使用

```sh
mmuuai catalog --kind model
mmuuai skills
mmuuai prompts --query 剧本
mmuuai quote mdl_REPLACE_WITH_ID --prompt "一只蓝色杯子"
mmuuai run mdl_REPLACE_WITH_ID --prompt "一只蓝色杯子"
mmuuai task tsk_REPLACE_WITH_ID
mmuuai result tsk_REPLACE_WITH_ID
mmuuai call mmuu_prompts --json-file arguments.json
```

模型和组合能力使用当前后台目录与版本，命令不硬编码模型名称或价格。由 Agent 读取配套 Skill 后按需求选择默认模型，用户指定模型时按用户选择；纯文字创作先读免费预设，由当前 Agent 写作。`quote` 只询价，不扣次数或算力。

`run` 询价并提交一次，再查询同一任务、领取并自动保存结果。结果默认保存在系统下载目录的 `MMUUAI/`，平铺命名与网页下载规则相同。保存失败时完整已领取结果留在当前连接独占的私有恢复目录，重新执行同一 `result` 优先补存，不再次领取或生成；成功后清理该恢复文件。请保护恢复目录中的私人内容。

`create` 可用 `--idempotency-key` 指定稳定幂等键；省略时生成一个并在提交前输出恢复回执。请求结果未知时保留该回执，查询原任务；不要换键重复提交。`reconciling` 保持原任务，不重新发起生成。

生成任务领取的正文和大媒体不直接输出到终端，终端返回本机文件路径及安全摘要。需要显示图片、播放音视频或读取文字时，由 Agent 使用本机文件。目录、预设及用户主动读取的资料仍按命令输出 JSON。`--json-file -` 支持从标准输入传入 JSON，避免把私人正文写入命令行历史。
