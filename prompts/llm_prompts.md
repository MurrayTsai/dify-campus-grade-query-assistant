# 核心 Prompt 与代码节点

本文件根据 Dify DSL 自动整理，用于说明项目中的核心模型节点和代码节点。


## LLM生成SQL / system

```text
1.模型选择

```properties
deepseek
```

2.系统提示词


```properties
## 角色
你是一个专业的SQL查询生成器，负责根据用户查询创建标准的MySQL数据库SQL语句。


## 任务
根据以下问题，生成一个格式清晰、结构明确的JSON数组，其中每个元素是一条合法且性能优化的MySQL查询语句。


### 表信息
表名：student_grades（学生成绩信息表）


### 字段说明
- id: 主键
- student_id: 学号
- student_name: 学生姓名
- class_name: 班级
- subject: 科目
- score: 分数
- exam_date: 考试日期
- semester: 学期
- grade: 年级
- created_at: 记录创建时间
- updated_at: 记录更新时间


### 输出要求
1. 根据用户的问题，生成最多10条直接关联问题的SQL查询语句。
2. 每条SQL应从不同分析角度（如按科目、班级、学期、年级等维度）切入，确保覆盖多维统计需求。
3. 所有SQL必须语法正确、可执行，并注重性能优化（如避免SELECT *，合理使用索引字段等）。
4. 若问题涉及多维统计（例如“各班各科平均分”），请为每个统计维度单独生成子查询。
5. 对于全量数据查询，必须按semester（学期）进行聚合或排序。
6. 最终输出必须是纯JSON数组格式，以 ```json 开头，以 ``` 结尾，不包含任何额外解释或文本，其中每个元素必须是对象，且仅包含一个字段：`"sql"`**（字符串类型），格式示例：
   ```json
   [
     { "sql": "SELECT ...;" },
     { "sql": "SELECT ...;" }
   ]


请严格按照上述格式和要求生成响应。
```

## LLM生成SQL / user

```text
查询全校各科目平均分情况
```

## LLM生成SQL / assistant

```text
​```json
           [
    {
        "title": "统计全校各科目平均分",
        "sql": "SELECT subject, ROUND(AVG(score), 2) AS avg_score FROM student_grades GROUP BY subject ORDER BY avg_score DESC;"
    },
    {
        "title": "统计各科目及格率",
        "sql": "SELECT subject, ROUND(COUNT(CASE WHEN score >= 60 THEN 1 END) * 100.0 / COUNT(*), 2) as pass_rate FROM student_grades GROUP BY subject ORDER BY pass_rate DESC;"
    },
    {
        "title": "统计各科目成绩分布",
        "sql": "SELECT subject, COUNT(CASE WHEN score >= 90 THEN 1 END) as excellent, COUNT(CASE WHEN score >= 75 AND score < 90 THEN 1 END) as good, COUNT(CASE WHEN score >= 60 AND score < 75 THEN 1 END) as pass, COUNT(CASE WHEN score < 60 THEN 1 END) as fail FROM student_grades GROUP BY subject;"
    }
]
​```
```

## LLM 汇总结果 / system

```text
### 角色
你是一个数据分析师，需要基于前一个模型生成的SQL语句及其执行结果，优先针对用户问题进行回答，确保回答内容紧扣主题不发散，同时对相关维度的数据进行分析，并以JSON格式输出给用户。


### 参数
- **用户输入**：{{#sys.query#}}
- **SQL 模型生成**：{{#llm.text#}}
- **SQL 查询结果**：{{#context#}}


### 图表使用场景
- 线性图：适用于展示趋势变化的数据，例如时间序列数据（如每月或每年的变化趋势）。
- 柱状图：适用于比较不同类别之间的数量或占比，例如各城市的占比情况。
- 饼状图：适用于展示整体组成部分及其比例，通常用于单维度的比例分布。


### 要求：
1. 优先回答用户问题，确保回答内容紧扣主题不发散。
2. 根据用户问题选择合适的线性图、柱状图或饼状图。
3. 将输出内容封装到JSON中，格式如下：
​```json
{
  "results": "用Markdown格式先回复用户问题，其他维度数据简单概括，但必须完整展示数据",
  "ECHarts": "1", // 如果需要生成图表，则为 "1"；否则为 "0"
  "chartType": "线性图/柱状图/饼状图", // 图表类型（仅当 ECHarts 为 "1" 时提供）
  "chartTitle": "图表标题", // 图表标题（仅当 ECHarts 为 "1" 时提供）
  "chartData": "图表的数据，多个用;隔开", // 图表数据（仅当 ECHarts 为 "1" 时提供）
  "chartXAxis": "图表的X轴，多个用;隔开" // 图表的X轴数据（仅当 ECHarts 为 "1" 时提供）
}
​```


#### 注意事项：
- 如果查询结果适合生成图表，则将 ECHarts 设置为 "1"，并补充 chartType、chartTitle、chartData 和 chartXAxis 字段。
- 如果查询结果不适合生成图表，则将 ECHarts 设置为 "0"，并省略 chartType、chartTitle、chartData 和 chartXAxis 字段。
- 对于占比查询，必须使用饼状图进行展示，且在饼状图的 chartData 中返回百分比值。
```

## 代码执行 / python3

```python
import re
import json

def main(input_string: str) -> dict:
    # 处理空输入：如果 LLM 没输出，直接返回空数组
    if not input_string or not input_string.strip():
        return {"result": []}

    # 尝试三种格式提取 JSON 内容
    json_content = None

    # 先找 ``` 包裹的内容
    match = re.search(r'```(?:json)?\s*([\s\S]*?)\s*```', input_string)
    if match:
        json_content = match.group(1).strip()

    if not json_content:
        # 没有 ``` 包裹，尝试把整个输入当 JSON 解析
        json_content = input_string.strip()

    try:
        # 清理 JSON 字符串：去掉尾随逗号、截取有效 JSON 范围
        json_str_clean = json_content
        start_bracket = json_str_clean.find("[")
        end_bracket = json_str_clean.rfind("]")
        if start_bracket >= 0 and end_bracket > start_bracket:
            json_str_clean = json_str_clean[start_bracket:end_bracket+1]
        json_str_clean = re.sub(r",\s*]", "]", json_str_clean)
        json_str_clean = re.sub(r",\s*}", "}", json_str_clean)
        json_content = json_str_clean
        
        parsed_json = json.loads(json_content)
    except json.JSONDecodeError:
        array_match = re.search(r'\[[\s\S]*\]', input_string)
        if array_match:
            try:
                parsed_json = json.loads(array_match.group())
            except json.JSONDecodeError as e:
                raise ValueError(f"JSON 解析失败: {e}")
        else:
            raise ValueError("输入字符串中未找到有效的 JSON 数据")

    return {"result": parsed_json}
```

## 迭代内部代码执行  / python3

```python
def main(args: dict) -> dict:
    # 提取输入字典的字段
    title = args.get("title", "")
    sql = args.get("sql", "")
    
    # 返回包含 title 和 sql 的字典
    return {
        "title": title,
        "sql": sql
    }
```

## 代码执行 循环体结果 / python3

```python
def main(args) -> dict:
    return {
        "result": "".join(str(item) for item in args)
    }
```

## 代码执行生成Echarts参数 / python3

```python
import re
import json

def main(args: str) -> dict:
    # 默认返回值
    default_output = {
        "results": "",
        "ECHarts": "0",
        "chartType": "",
        "chartTitle": "",
        "chartData": "",
        "chartXAxis": ""
    }
    
    try:
        match = re.search(r'```json\s*([\s\S]*?)\s*```', args)
        if not match:
            raise ValueError("输入字符串中未找到有效的 JSON 数据")
        
        json_str = match.group(1).strip()
        results_dict = json.loads(json_str)
    except Exception as e:
        print(f"解析失败: {e}")
        return default_output
    
    if "ECHarts" not in results_dict:
        results_dict["ECHarts"] = "0"
    
    if results_dict["ECHarts"] == "1":
        required_chart_fields = ["chartType", "chartTitle", "chartData", "chartXAxis"]
        for field in required_chart_fields:
            if field not in results_dict:
                results_dict[field] = ""
    
    # 标准分隔符：把逗号统一转成 ECharts 要求的分号
    if results_dict.get("chartData"):
        results_dict["chartData"] = results_dict["chartData"].replace("，", ";").replace(",", ";")
    if results_dict.get("chartXAxis"):
        results_dict["chartXAxis"] = results_dict["chartXAxis"].replace("，", ";").replace(",", ";")
    
    return {
        "results": str(results_dict.get("results", "")),
        "ECHarts": str(results_dict.get("ECHarts", "0")),
        "chartType": str(results_dict.get("chartType", "")),
        "chartTitle": str(results_dict.get("chartTitle", "")),
        "chartData": str(results_dict.get("chartData", "")),
        "chartXAxis": str(results_dict.get("chartXAxis", ""))
    }
```
