# Test Case 01：同名冲突

## 输入模板

- `target_name`：
- `alt_names`：
- `known_anchor`：
- `goal`：判断是否值得优先 sourcing

## 预期重点

- 先列候选身份，不要直接合并
- 至少找到 2 个一致锚点再合并资料
- 明确输出 `match confidence`、`false-match risk`、`evidence density`
- 如果身份仍不稳，只给候选判断和下一步验证建议
