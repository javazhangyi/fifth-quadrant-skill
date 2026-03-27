# fq@Feishu 文档评论统计

Skill 标识：fq@feishu-doc-comment-stats

## 1. 功能描述
该 Skill 用于统计指定 Feishu 文档中的评论数量，并提取每条评论的评分（如“优秀”“合格”）及评论时间。可用于教师作业打分、文档评审统计等场景。

---

## 2. 前置条件
1. 已拥有 Feishu 开发者账号，并创建了应用（获取 `app_id` 和 `app_secret`）。
2. 已获取 Feishu 文档的**文档 id**（file token）。
3. 已安装 `curl` 或支持 HTTP 请求的环境（可选安装 `jq` 用于解析 JSON）。
4. 建议将敏感信息放在 `.env` 中加载（避免在命令行历史/脚本中明文出现）。

### 2.1 通过 `.env` 提供 APP_ID / APP_SECRET（推荐）
创建 `.env` 文件（**不要提交到 git**）：

```bash
# .env
APP_ID=cli_xxx
APP_SECRET=xxx
FILE_TOKEN=doccnxxxx
FILE_TYPE=docx
TZ=Asia/Shanghai
```

在当前 shell 会话中加载 `.env`（zsh/bash 通用）：

```bash
set -a
source .env
set +a
```

安全要求：
- `.env` 必须加入 `.gitignore`
- 任何日志/截图/输出中不要打印 `APP_SECRET`、`tenant_access_token`

---

## 3. 流程步骤

### 3.1 获取 Tenant Access Token
请求（从 `.env` 读取 `APP_ID`/`APP_SECRET`）：

```bash
curl --location 'https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal' \
  --header 'Content-Type: application/json; charset=utf-8' \
  --data "{
    \"app_id\": \"${APP_ID}\",
    \"app_secret\": \"${APP_SECRET}\"
  }"
```

示例返回：

```json
{
  "code": 0,
  "expire": 7200,
  "msg": "ok",
  "tenant_access_token": "t-g1043dkOZAUT6G4Y5CNR5E4OYTFNGHBEGR5IFO5R"
}
```

说明：Token 有效期 7200 秒，过期后需重新获取。

---

### 3.2 请求文档评论列表
请求：

```bash
curl --location "https://open.feishu.cn/open-apis/drive/v1/files/${FILE_TOKEN}/comments?file_type=${FILE_TYPE:-docx}" \
  --header "Authorization: Bearer ${TENANT_ACCESS_TOKEN}" \
  --header "Content-Type: application/json"
```

示例返回（部分）：

```json
{
  "code": 0,
  "data": {
    "has_more": false,
    "items": [
      {
        "comment_id": "7612829559754394805",
        "create_time": 1772500006,
        "quote": "3月2日",
        "reply_list": {
          "replies": [
            {
              "content": {
                "elements": [
                  { "text_run": { "text": "合格" } }
                ]
              },
              "create_time": 1772500006
            }
          ]
        }
      }
    ]
  },
  "msg": "Success"
}
```

分页：
- 当 `data.has_more=true` 时，使用 `page_token` 继续请求下一页：

```bash
curl --location "https://open.feishu.cn/open-apis/drive/v1/files/${FILE_TOKEN}/comments?file_type=${FILE_TYPE:-docx}&page_token=${PAGE_TOKEN}" \
  --header "Authorization: Bearer ${TENANT_ACCESS_TOKEN}" \
  --header "Content-Type: application/json"
```

---

### 3.3 数据解析
1. 评论数量：`data.items` 数组长度（所有分页累计）。
2. 每条评论信息：
   - `quote`：评论引用内容（如“作业”或日期）
   - `create_time`：评论时间（Unix 时间戳，可转为可读格式）
   - `reply_list.replies[*].content.elements[*].text_run.text`：评论得分（如“优秀”“合格”）

---

### 3.4 示例统计输出
暂时无法在飞书文档外展示此内容。  
注：Unix 时间戳需转换为本地时间。

---

### 3.5 注意事项
1. Token 过期后需重新请求。
2. 评论可能存在多条回复，需遍历 `reply_list.replies`。
3. 文档类型需指定 `file_type=docx`（或与文档类型匹配）。
4. API 分页：`has_more=true` 时需使用 `page_token` 继续请求。

---

## 4. 扩展功能建议
- 可将统计结果导出为 CSV 或 Excel，便于进一步分析。
- 可按评分分类统计优秀率、合格率。
- 可结合 Feishu 用户信息，统计每位员工或学生的平均得分。

