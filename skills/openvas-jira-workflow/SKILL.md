# OpenVAS to Jira Issue Workflow

## 触发条件

用户请求以下任一操作时触发此 skill：
- "创建 OpenVAS 报告 Jira issue"
- "创建漏洞扫描 issue"
- "OpenVAS 转 Jira"
- 任何提及将漏洞扫描结果转换为 Jira issue 的请求

## 执行流程

### 步骤 1: 验证工具状态
```bash
openvas-cli doctor
jira version
```

### 步骤 2: 获取报告列表
```bash
openvas-cli report list
```

### 步骤 3: 获取用户输入
- **必填**: Jira 项目 Key (如 `ENGR`)
- **必填**: 分配用户 (如 `林提祥`)
- **可选**: Issue 类型 (默认 `Story`)
- **可选**: 优先级 (默认 `High`)

### 步骤 4: 导出报告
```bash
openvas-cli report get --id <REPORT_ID> --format xml --output /tmp/openvas_report.xml
```

### 步骤 5: 分析漏洞统计
提取 Critical/High/Medium/Low/Log 数量用于摘要

### 步骤 6: 创建 Jira Issue
```bash
jira issue create -p<PROJECT> -tStory \
  -s"OpenVAS 漏洞扫描报告 - <扫描任务> (<漏洞统计>)" \
  -yHigh -b"<描述>" --no-input
```

### 步骤 7: 分配用户
```bash
jira issue assign <ISSUE-KEY> "<用户名>"
```

### 步骤 8: 返回结果
- Issue URL
- 漏洞统计摘要
- 提醒手动上传附件 (如 REST API 失败)

## 已知问题

1. Issue type `-tTask` 参数不工作，需使用 `-tStory`
2. 附件上传可能被代理阻挡，需手动在 Web 界面上传

## 文件位置

工作流文档: `workflows/openvas-jira-workflow.md`