---
name: fq@feishu-auto-grade-homework
description: 基于飞书 Wiki/多维表格/群消息，自动批改「上行」每日作业并记录得分、发优秀到群。使用 Lark-MCP 获取今日日期、读取总文档与各人子文档、评分（优秀/合格/不合格）、留言、维护多维表格、向「上行2026实践小组」推送优秀作业与点评。适用于用户说「执行今日作业批改」「作业批改」「批改作业」或定时触发时。
---

# fq@自动批改作业

Skill 标识：fq@auto-grade-homework

通过 Lark-MCP 完成：获取今日作业 → 评分与选优 → 记录到多维表格 → 优秀作业发群并点评。每日可运行两次（如 19:00、20:00），未提交者发提醒。

## 前置条件

- 已配置 **Lark-MCP**，且机器人已加入目标群「上行2026实践小组」。
- **总文档**（含各人子文档入口）：<https://ln25hbn0u6.feishu.cn/wiki/PE1VwSax7it4P0kiR41cbZQMn0Z>  
  - Wiki 节点 token：`PE1VwSax7it4P0kiR41cbZQMn0Z`（总文档标题示例：「2026上行作业提交5人组」）。
- **目标群** chat_id：`oc_819a92bda6bd09f92f35845b7f6517ec`（上行2026实践小组）。

### 成员作业文档链接速查

- 孙嘉兴：<https://ln25hbn0u6.feishu.cn/docx/CX04dz27IomjJUxsatFc4Yhmnqe>
- 赵慧娟：<https://ln25hbn0u6.feishu.cn/wiki/FAmcwb2kUiteOBkq3LrcfM7RnBf>
- 李雪飞：<https://ln25hbn0u6.feishu.cn/wiki/Tgh2w97FFip383kBDxec4AuXndc>
- 李晓静：<https://ln25hbn0u6.feishu.cn/docx/FhWCdMQY0oYDI0xsEqkc81MdnAg>
- 张益：<https://ln25hbn0u6.feishu.cn/wiki/RfMBwsaYqiu0ptk4OO5c60EFnpg>

## 执行流程（按序执行）

```
进度清单：
- [ ] 1. 获取今日日期
- [ ] 2. 从总文档解析当月各人作业文档并拉取今日作业内容
- [ ] 3. 对每份作业评分并选出 1～2 份优秀
- [ ] 4. 为每份作业生成评语（留言）
- [ ] 5. 创建或更新多维表格，记录每人当日得分
- [ ] 6. 将今日优秀作业发到群并附简短点评
- [ ] 7.（可选）若有未提交者，发送提醒
```

### 步骤 1：获取今日日期

- 使用当前系统日期（或用户/调度传入的作业日期）。
- 得到年月日与月份用于匹配总文档和子文档中的当日作业。

### 步骤 2：获取各人今日作业内容

1. 调用 `wiki_v2_space_getNode` 获取总文档 `obj_token`。  
2. 调用 `docx_v1_document_rawContent` 拉取总文档正文。  
3. 解析当月成员与子文档映射。  
4. 用 `wiki_v1_node_search` 查子文档节点，再 `getNode + document_rawContent` 拉正文。  
5. 按“今日”或“X月X日”提取当天作业；无则记为未提交。

### 步骤 3：评分与选优

- 评分等级：优秀 / 合格 / 不合格。
- 从“优秀”中选 1～2 份作为当日优秀作业。

### 步骤 4：生成评语（留言）

- 每份作业生成 1～3 句评语。
- 若不支持写评论，则把评语落到多维表格“评语”列。

### 步骤 5：创建或更新多维表格

- 新建或复用 Base（如“上行2026-每日作业得分”）。
- 建议字段：日期、姓名、得分、评语、是否今日优秀。
- 用 `bitable_v1_appTableRecord_create` 写入；同人同日仅保留一条。

### 步骤 6：优秀作业发群

- 群：`oc_819a92bda6bd09f92f35845b7f6517ec`，`receive_id_type=chat_id`。
- 内容：标题 + 1～2 份优秀作业成员名与链接 + 简短点评。
- 接口：`im_v1_message_create`。

### 步骤 7：未提交提醒（可选）

- 有未提交名单时，在同一群发送提醒。
- 全员已交则跳过。

## 评分与评语模板

- 优秀：观点清晰，有具体例子，结构完整，继续保持。  
- 合格：已按要求完成，建议下次增加自己的思考。  
- 不合格：本次未看到当日作业内容，请补交或下次按时提交。  

## 定时与触发

- 单次执行流程；建议每日 19:00、20:00 各执行一次（外部调度）。
- 触发语示例：执行今日作业批改 / 作业批改 / 批改作业。

## 授权过期处理

- 若出现授权过期，重新登录 Lark-MCP（请使用自己的密钥，不要写入文档）：

```bash
npx -y @larksuiteoapi/lark-mcp login -a "${APP_ID}" -s "${APP_SECRET}"
```

## 关键 Lark-MCP 接口速查

- `wiki_v2_space_getNode`
- `docx_v1_document_rawContent`
- `wiki_v1_node_search`
- `bitable_v1_app_create`
- `bitable_v1_appTable_create`
- `bitable_v1_appTableRecord_create`
- `bitable_v1_appTableRecord_search`
- `im_v1_message_create`

## 快捷用法：只检查未交并提醒

- 当用户只要求“检查未交+提醒”时，仅执行步骤 1～2 + 7，跳过 3～6。

