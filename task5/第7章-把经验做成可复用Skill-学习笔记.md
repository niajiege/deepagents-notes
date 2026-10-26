# 第7章：把经验做成可复用 Skill

## 一、本章任务

第7章讲 Skill：把「一件事该怎么做」从提示词里搬出来，做成一个可以被按需加载的目录。

作业要求是在主项目的 `skills` 目录里改造或新增一种内容流程。我新增了一个 `material-report` Skill，专门负责「用 `/materials/` 下的固定资料写对比报告」，然后把提示词里那一大段报告格式规则换成了指向它的一句话。


## 二、第7章讲的是什么

Skill 本身很简单：**一个目录 + 一个 `SKILL.md`**。

- `SKILL.md` 开头是 YAML frontmatter，必填 `name` 和 `description`；后面是 Markdown 正文，写具体步骤。
- 目录里还可以放 `references/`（参考资料）、`assets/`（模板之类）、`scripts/`（脚本）。

它和提示词最大的区别是**不在上下文里常驻**。官方把这叫渐进披露，分三层：

| 层 | 内容 | 什么时候进上下文 |
|---|---|---|
| Level 1 | 每个 Skill 的 name、description、路径 | 启动时，全部写进 system prompt |
| Level 2 | `SKILL.md` 正文 | 模型决定用了，用 `read_file` 读 |
| Level 3 | `references/`、`assets/`、`scripts/` | 正文里提到、且真的需要时再读 |

一句话：**description 决定「要不要用」，正文决定「怎么用」，资源文件决定「细节」。**

## 三、我加的 Skill

```
workspace/skills/material-report/
├── SKILL.md
├── references/report-format.md
└── assets/report-template.md
```

- **SKILL.md**：description 里同时写了「什么时候用」和「什么时候不用」——基于 `/materials/` 写对比报告、汇总报告时用；需要联网检索的任务不要用。正文五步：先看清单 → 逐份提炼 → 按格式写 → 落盘 → 读回核对。
- **references/report-format.md**：报告结构、对比表的五列硬性要求（是否默认启用／注入什么／在图中的位置／关键参数／已知限制）、引用格式、长度上限。
- **assets/report-template.md**：一份空白骨架，五列表头都填好了，照着填就行。

## 四、代码改了三处

1. **`agent.py` 注册 Skill**

   ```python
   skills=["/skills/"],
   ```

   这里踩了一下：路径是相对 **backend 根目录**的，不是相对仓库根目录。`FilesystemBackend` 的 `root_dir` 是 `workspace/`，所以 `/skills/` 实际指向 `workspace/skills/`。以后改 `root_dir`，Skill 的位置也跟着变。

2. **`prompts.py` 第 5 步改成一句话**

   ```
   5. **Write Report**: Write a comprehensive final report to `/final_report.md`.
      The report format rules live in the `material-report` skill, not in this
      prompt: read `/skills/material-report/SKILL.md` before writing.
   ```

   同时把原来那整段 `## Report Writing Guidelines`（1561 字符）删掉了。格式规则原来只在提示词里，现在只在 Skill 里。

3. **`.gitignore` 加例外**

   `workspace/*` 是全忽略的，不加例外的话 Skill 进不了 git，别人 clone 下来就没有这个能力：

   ```
   !workspace/materials/
   !workspace/skills/
   ```

## 五、验证一：正文到底进没进 prompt

写了个小脚本 `scripts/check_skills.py`，用一个只记录、不真的发请求的模型替换掉真模型，把第一次请求的 system prompt 抓下来看：

```
system prompt chars: 8004
description from SKILL.md frontmatter: True
SKILL.md body reached the prompt:      False
references/ reached the prompt:        False
```

- description（含「把多份文档整理成一张对比表」这句）出现在 prompt 里 —— Level 1 进来了。
- 正文里的「先看清单」、references 里的「对比表的硬性要求」都没出现 —— Level 2、3 没有提前进。

这就是渐进披露该有的样子。system prompt 一共 8004 字符，比原来少了一截，因为删了 1561 字的格式段，只加了一句指向 Skill 的话。

## 六、验证二：真跑一遍

为了不覆盖原来的 `workspace/final_report.md`，我临时复制了一份 workspace 当 backend 根目录来跑，跑完删掉。工具调用的顺序是这样的：

```
read_file  /skills/material-report/SKILL.md            limit=1000   ← 第一件事
ls         /materials
read_file  /materials/01-filesystem-middleware.md      limit=1000
read_file  /materials/02-todolist-middleware.md        limit=1000
read_file  /materials/03-summarization-middleware.md   limit=1000
read_file  /skills/material-report/references/report-format.md
read_file  /skills/material-report/assets/report-template.md
write_file /final_report.md
read_file  /final_report.md
```

几个观察：

- **读 Skill 是第一件事**，在 `ls /materials` 之前。也就是说它先读了说明，再按说明去干活。
- 读 `SKILL.md` 和读资料都带了 `limit=1000`。因为我在 SKILL.md 的「注意事项」里写了 `read_file` 默认只读 100 行，长文件要传 `limit`，它照做了。
- `references/` 和 `assets/` 是读完正文才去读的 —— Level 3 确实按需加载。
- 报告 59 行、5888 字节，三份资料都用上了，对比表按 references 要求的五列写的（它自己还多加了一行「数据落点」，不影响）。全程没有 `tavily_search`，因为这次任务只用固定资料。

## 七、几个坑

- **Skill 路径相对 backend 根目录**，不是相对仓库根。写 `/workspace/skills/` 会找不到。
- **子 Agent 不继承主 Agent 的 skills**（第5章那条规则：`system_prompt`、`middleware`、`skills` 都不继承）。想让子 Agent 也用这个 Skill，得在子 Agent 的 dict 里单独写一份。
- **`read_file` 默认只读 100 行**，长文件要传 `limit`。这条我写进了 SKILL.md 里提醒模型。
- **格式规则删掉以后，报告质量全靠 Skill 撑着。** Skill 读不到（路径写错、没进 git），报告就会退回没有格式要求的状态 —— 所以 `.gitignore` 那条例外不是顺手加的，是必需的。

## 八、本章完成结果

- [✅] 新增 `workspace/skills/material-report/`（SKILL.md + references + assets）
- [✅] `agent.py` 注册 `skills=["/skills/"]`
- [✅] `prompts.py` 第 5 步指向 Skill，删掉 1561 字的格式段
- [✅] `.gitignore` 给 `workspace/skills/` 开例外
- [✅] 离线验证：只有 description 进 system prompt，正文和 references 都没进
- [✅] 真跑验证：三层按顺序加载，报告正常产出
- [ ] 第6章异步子 Agent（选做，没做）

## 参考

- [第7章：Skills](https://datawhalechina.github.io/deepagents-in-action/chapters/ch07-skills/)
- [第6章：异步子 Agent](https://datawhalechina.github.io/deepagents-in-action/chapters/ch06-async-subagents/)
