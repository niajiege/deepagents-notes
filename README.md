# deepagents-notes

《Deep Agents 实战》课程学习笔记 —— Datawhale 开源课程
[datawhalechina/deepagents-in-action](https://github.com/datawhalechina/deepagents-in-action)

## 目录

| Task | 笔记 | 内容 |
|---|---|---|
| task0 | [AgentSeek 准备篇](task0/AgentSeek准备篇-学习笔记.md) | 环境准备、`create`/`info`/`task`/`doctor`/`dev` 命令链、`.env` 配置、预防性踩坑 |
| task1 | [第1-2章：我的第一个工具 Agent](task1/第1-2章-我的第一个工具Agent-学习笔记.md) | 启动主应用、Agent 执行循环、添加自定义工具 `get_weather`（高德天气）、真实踩坑记录 |
| task2 | [第3章：让成果留在文件里](task2/第3章-让成果留在文件里-学习笔记.md) | 虚拟文件系统、换用 `FilesystemBackend` 让产物落盘、保存→读回→修改、自动上下文管理（大结果卸载＋对话摘要）、真实踩坑记录 |

## 环境

- 系统：WSL2 Ubuntu（AgentSeek 项目）＋ Windows 原生（课程 Notebook）
- Python 3.12 ／ uv ／ Node.js LTS
- AgentSeek `0.1.2`（教程 2026-08-18 验证，实际模板为 main 分支更新版）

## 说明

笔记为 Markdown，截图与本目录内 `.md` 同级存放，图片使用相对路径引用。
