# 数据库表结构说明

本项目默认使用 MySQL 数据库，核心表名为：

```sql
student_grades
```

## 字段说明

| 字段 | 类型建议 | 说明 |
|---|---|---|
| id | BIGINT / INT | 主键 |
| student_id | VARCHAR(50) | 学号 |
| student_name | VARCHAR(100) | 学生姓名 |
| class_name | VARCHAR(100) | 班级 |
| subject | VARCHAR(100) | 科目 |
| score | DECIMAL(5,2) | 分数 |
| exam_date | DATE | 考试日期 |
| semester | VARCHAR(50) | 学期 |
| grade | VARCHAR(50) | 年级 |
| created_at | DATETIME | 记录创建时间 |
| updated_at | DATETIME | 记录更新时间 |

## 建表示例

```sql
CREATE TABLE student_grades (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  student_id VARCHAR(50) NOT NULL,
  student_name VARCHAR(100) NOT NULL,
  class_name VARCHAR(100) NOT NULL,
  subject VARCHAR(100) NOT NULL,
  score DECIMAL(5,2) NOT NULL,
  exam_date DATE,
  semester VARCHAR(50),
  grade VARCHAR(50),
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

## 索引建议

为了提升查询性能，可以添加以下索引：

```sql
CREATE INDEX idx_student_id ON student_grades(student_id);
CREATE INDEX idx_student_name ON student_grades(student_name);
CREATE INDEX idx_class_name ON student_grades(class_name);
CREATE INDEX idx_subject ON student_grades(subject);
CREATE INDEX idx_semester ON student_grades(semester);
CREATE INDEX idx_grade ON student_grades(grade);
```

## 数据安全建议

如果项目用于真实校园环境，建议：

1. 使用脱敏学生姓名和学号。
2. 数据库账号只授予 `SELECT` 权限。
3. 不允许 LLM 生成修改数据的 SQL。
4. 对输出结果做权限控制，避免普通用户查询他人成绩。
