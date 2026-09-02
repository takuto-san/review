---
translation_of: skills/maintain-review-plugin/SKILL.md
language: zh-CN
runtime: false
---

# 维护Review Plugin

当`agents/`、`skills/`或`.claude-plugin/`中的文件被添加、删除、重命名或发生行为变更时，保持运行时定义、manifest、日语和简体中文文档同步。

英文运行时定义是正本。除非用户明确要求行为变更，否则保留现有职责和策略。对应翻译必须同步标题、必需输入、边界、完成条件、标识符和输出字段。安装、使用方式、架构、行为或自定义指南发生变化时，同时更新三种语言的根README。

完成前检查Skill引用的所有代理都存在并已在manifest注册，`review_item_id`可追踪，JSON和frontmatter可解析，并运行`git diff --check`。Claude CLI可用时还要运行`claude plugin validate . --strict`。不要翻译代码标识符、路径、枚举值、YAML键、命令或代理名称。
