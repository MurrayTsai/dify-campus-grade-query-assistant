# 项目结构说明

```text
dify-campus-grade-query-assistant/
├─ README.md
├─ .gitignore
├─ dify/
│  └─ workflow.yml
├─ docs/
│  ├─ import_to_dify.md
│  ├─ workflow_overview.md
│  ├─ database_schema.md
│  ├─ project_structure.md
│  └─ security_notes.md
├─ prompts/
│  └─ llm_prompts.md
├─ test_cases/
│  └─ evaluation_cases.md
├─ data/
│  └─ README.md
└─ screenshots/
   └─ README.md
```

## 文件说明

| 路径 | 说明 |
|---|---|
| README.md | 项目首页说明，适合 GitHub 展示 |
| dify/workflow.yml | 从 Dify 导出的 DSL 文件，可重新导入 Dify |
| docs/import_to_dify.md | 导入 Dify 的操作步骤 |
| docs/workflow_overview.md | 工作流节点和流程说明 |
| docs/database_schema.md | 数据库表结构说明 |
| docs/security_notes.md | 数据安全和 SQL 安全说明 |
| prompts/llm_prompts.md | 工作流中的核心 Prompt 和代码节点说明 |
| test_cases/evaluation_cases.md | 测试问题和预期结果 |
| data/README.md | 数据集说明，不包含真实学生数据 |
| screenshots/README.md | 放置工作流截图的位置说明 |
