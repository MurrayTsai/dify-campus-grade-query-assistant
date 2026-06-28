# Dify 校园数据库查询助手

## 项目简介

本项目是一个基于 **Dify Workflow** 搭建的校园成绩数据库查询助手。用户可以用自然语言提出成绩查询问题，系统会自动生成 MySQL 查询语句，调用数据库执行查询，并将查询结果汇总成 Markdown 文本；当结果适合可视化时，还会调用 ECharts 工具生成折线图、柱状图或饼图。

- 应用名称：校园数据库查询
- 应用描述：用于校园成绩快速查询
- 应用模式：advanced-chat
- Dify DSL 版本：0.6.0

## 适用场景

- 学生个人成绩查询
- 班级成绩统计
- 科目平均分统计
- 年级/学期成绩对比
- 及格率、优秀率、成绩分布分析
- 查询结果图表化展示

## 核心能力

1. **自然语言转 SQL**：将用户问题转换为可执行的 MySQL 查询。
2. **数据库查询执行**：通过 Dify Database 插件执行 SQL。
3. **结果汇总分析**：由 LLM 根据 SQL 结果生成可读的分析结论。
4. **图表生成**：根据数据类型自动选择线性图、柱状图或饼图。
5. **结构化输出**：通过 JSON 约束模型输出，方便后续节点解析。

## 项目结构

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

## 工作流概览

```text
用户输入
  ↓
LLM 生成 SQL
  ↓
解析 SQL JSON
  ↓
循环执行 SQL
  ↓
汇总 SQL 执行结果
  ↓
LLM 分析结果并判断是否需要图表
  ↓
解析 ECharts 参数
  ↓
条件分支：折线图 / 柱状图 / 饼图 / 纯文本
  ↓
返回最终结果
```

## 数据库表结构

工作流中的核心表为：`student_grades`

| 字段 | 含义 |
|---|---|
| id | 主键 |
| student_id | 学号 |
| student_name | 学生姓名 |
| class_name | 班级 |
| subject | 科目 |
| score | 分数 |
| exam_date | 考试日期 |
| semester | 学期 |
| grade | 年级 |
| created_at | 记录创建时间 |
| updated_at | 记录更新时间 |

## 依赖插件

- `langgenius/deepseek:0.0.17@f7a5c1d222d17ee419b17a80d6e4923838fe93788dc724a3af73fbc6990b3a32`
- `langgenius/echarts:0.0.7@1a9050c16b63a1c82bd1632c00fa495af4d8854d25fb7d465d47775e1b798eff`
- `hjlarry/database:0.0.6@534bc26cf5bc4ff6b5557457452287ccc71f00eef9378784c4f43ca49954ca2f`

## 导入 Dify

1. 打开 Dify。
2. 进入 **Studio / 工作室**。
3. 选择 **Import DSL / 导入 DSL**。
4. 上传 `dify/workflow.yml`。
5. 重新配置模型供应商和插件授权。
6. 配置 Database 插件的数据库连接。
7. 运行测试问题，确认 SQL 生成、查询执行和图表展示正常。

更详细步骤见：[`docs/import_to_dify.md`](docs/import_to_dify.md)

## 测试问题示例

```text
查询全校各科目平均分情况
查询三年级各班数学平均分
查询本学期各科及格率
查询张三的所有科目成绩
统计各班优秀人数
展示各科平均分柱状图
查询不同学期数学平均分变化趋势
```

完整测试集见：[`test_cases/evaluation_cases.md`](test_cases/evaluation_cases.md)

## 注意事项

- 本仓库只保存 Dify DSL 和项目文档，不保存真实数据库账号、密码、API Key。
- `workflow.yml` 中 Database 插件的 `db_uri` 为空，需要导入 Dify 后重新配置。
- 如果用于真实校园场景，请务必脱敏学生姓名、学号、成绩等隐私数据。
- 当前 SQL 生成节点需要继续强化安全约束，建议限制只允许 `SELECT` 查询，禁止 `INSERT`、`UPDATE`、`DELETE`、`DROP` 等危险语句。

## 项目价值

这个项目可以作为 AI 产品经理 / AI 应用开发作品集，展示以下能力：

- Dify 工作流搭建
- LLM 自然语言转 SQL 设计
- 数据查询类 Agent 流程设计
- Prompt 结构化输出设计
- 数据分析与图表可视化
- 测试用例设计与项目文档整理
