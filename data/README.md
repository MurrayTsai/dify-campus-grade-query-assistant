# 数据说明

本项目不包含真实学生成绩数据。

为了保护隐私，GitHub 仓库中不要上传真实学生姓名、学号、成绩等信息。

如果需要演示，可以自行准备模拟数据，字段与 `docs/database_schema.md` 中的 `student_grades` 表保持一致。

建议使用模拟数据，例如：

```sql
INSERT INTO student_grades
(student_id, student_name, class_name, subject, score, exam_date, semester, grade)
VALUES
('S2024001', '学生A', '一班', '数学', 92, '2026-06-01', '2026春季', '三年级');
```
