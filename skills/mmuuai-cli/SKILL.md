---
name: mmuuai-cli
description: 通过 MMUUAI CLI 生成处理图片、视频、音乐、语音，下载解析视频和采集视频案例；写文章、剧本、歌词、广告文案和提示词时，主动读取免费公开预设与平台知识，由当前 Agent 创作。支持查询平台资源和获准访问的本人资料，优先复用已加载的原生工具；普通问答和代码工作不必调用。
---

# MMUUAI CLI

这是独立命令行入口。使用前必须读取 [平台使用规则](references/platform-rules.md)，该文件在打包时从插件主 Skill 生成；本文件只补充命令行操作，不另写一套创作、权限或提问规则。

选定非语言模型、技能或工作流后，按平台规则先取得同资源、同版本的 `usageGuide/usageContract`，再准备问题与请求。本轮已完整读取则复用；读到规范不等于用户已批准执行。

将平台规则中的每个 `mmuu_*` 工具调用转换为：

```sh
mmuuai call mmuu_catalog --json-file arguments.json
```

先运行 `mmuuai --help` 确认当前命令。参数 JSON 原样传递，也可通过 `--json-file -` 从标准输入读取。优先使用 `catalog`、`quote`、`create`、`wait`、`confirmation`、`confirm`、`result` 等简短命令。

- 没有登录时让用户执行 `mmuuai login`，在 MMUUAI 官方授权页登录；不索取密码、Cookie 或 Token。
- `catalog --kind model|skill|workflow --query 关键词` 搜索；用上述 `call mmuu_catalog` 传 `{kind,resourceId}` 读取详情。其他资料与资产工具按同样方式调用，参数以实际工具 Schema 为准。
- `quote` 准备最终清单，用户批准同版本、同输入后才 `create`。工作流顶层 `workflowMode` 使用用户所选的 `step_by_step/automatic`；独立模型不加工作流模式。
- `wait` 遇到 `awaiting_confirmation` 时已返回下一步完整清单；按平台规则展示并取得本步批准后 `confirm`，再 `wait` 原任务。回执完整时不重复查询或保存前步；不循环自动批准。
- `run` 仅用于已经批准的完整请求，已包含首次结果领取与自动保存；不要随后再调用首次领取工具。直接调用 `mmuu_get_result` 也由 CLI 保存并返回文件路径。
- 保存失败时优先对同一任务执行 `result` 补存。请求未知或 `reconciling` 时查询原任务，不重新生成，不自动换幂等键。
- Python 可通过 `MMUUAI_PYTHON` 指定真实绝对路径。若返回 `MMUU_CLI_UPLOAD_HELPER_FILE_UNAVAILABLE` 且 `uploadAttempted=false`，该 Python 看不到安装目录，尚未发送文件；用宿主依赖工具提供的可用 Python 配置后再上传。保存器出现 `MMUU_SAVE_BRIDGE_HELPER_FILE_UNAVAILABLE` 时同样切换 Python，再对原任务 `result` 补存，不重复领取或生成。只在当前 CLI 进程环境设置，不改全局 Python 或账号授权。
- 结果默认自动保存到系统下载目录/MMUUAI/，用户指定其他目录时使用 `--output-dir`。通过 CLI 回执中的绝对路径展示媒体或读取文字，避免将大 base64 放进对话。

不要把当前软件不支持的插件或 Skill 安装方式说成已完成；CLI 连接不等于宿主已经加载了原生 MCP 工具。
