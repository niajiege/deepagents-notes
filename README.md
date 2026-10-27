# deepagents-notes

《Deep Agents 实战》课程学习笔记 —— Datawhale 开源课程
[datawhalechina/deepagents-in-action](https://github.com/datawhalechina/deepagents-in-action)

## 目录

| Task | 笔记 | 内容 |
|---|---|---|
| task0 | [AgentSeek 准备篇](task0/AgentSeek准备篇-学习笔记.md) | 环境准备、`create`/`info`/`task`/`doctor`/`dev` 命令链、`.env` 配置、预防性踩坑 |
| task1 | [第1-2章：我的第一个工具 Agent](task1/第1-2章-我的第一个工具Agent-学习笔记.md) | 启动主应用、Agent 执行循环、添加自定义工具 `get_weather`（高德天气）、真实踩坑记录 |
| task2 | [第3章：让成果留在文件里](task2/第3章-让成果留在文件里-学习笔记.md) | 虚拟文件系统、换用 `FilesystemBackend` 让产物落盘、保存→读回→修改、自动上下文管理（大结果卸载＋对话摘要）、真实踩坑记录 |
| task3 | [第4章：让项目按计划完成](task3/第4章-让项目按计划完成-学习笔记.md) | `TodoListMiddleware` 的作用与图结构变化（6/7 对 5/6、工具 14 对 13）、固定资料多步骤任务、清单全绿≠产物正确的核对、两个真实问题 |
| task4 | [第5章：把工作交给子 Agent](task4/第5章-把工作交给子Agent-学习笔记.md) | 上下文隔离（只回结论不回过程）、三个必填字段与继承规则、项目现状摸底（14 个工具、2 个可委派对象）、改造方案（职责／工具／提示词） |
| task5 | [第7章：把经验做成可复用 Skill](task5/第7章-把经验做成可复用Skill-学习笔记.md) | Skill 目录结构与三层渐进披露、新增 `material-report` Skill（SKILL.md＋references＋assets）、提示词减 1561 字、两次验证（只有 description 进 prompt／真跑时三层按顺序加载） |
| task6 | [第8-9章：记住偏好，执行前先审批](task6/第8-9章-记住偏好与执行前审批-学习笔记.md) | 长期记忆（Store＋两条路由、个人/全局两种作用域、预置记忆文件）、HITL（`interrupt_on`、检查点、approve/edit/reject）、前端审批卡片、六张真机截图、两个验证脚本，以及"偏好被写进全局"的坑 |

## 环境

- 系统：WSL2 Ubuntu（AgentSeek 项目）＋ Windows 原生（课程 Notebook）
- Python 3.12 ／ uv ／ Node.js LTS
- AgentSeek `0.1.2`（教程 2026-08-18 验证，实际模板为 main 分支更新版）

## 说明

笔记为 Markdown，截图与本目录内 `.md` 同级存放，图片使用相对路径引用。
