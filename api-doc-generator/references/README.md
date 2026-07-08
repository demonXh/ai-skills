# api-doc-generator 参考资源

本目录提供 `api-doc-generator` skill 的配套模板与样例。

| 文件 | 用途 |
|------|------|
| `doc-template.md` | 5 章节标准空白模板 |
| `http-api-example.md` | HTTP 接口完整范例（含 4 环境 URL 与流程图占位） |
| `checklist.md` | 文档完成度自查清单 |

## 真实文档样本

- `docs/api/OfficePrinterClient.md` - 项目内首份按 5 章节规范产出的文档

## 使用顺序

1. 阅读上级目录的 `SKILL.md` 了解规范与工作流
2. 复制 `doc-template.md` 到 `docs/api/<业务名>.md`
3. 参照 `http-api-example.md` 填充内容
4. 调用 excalidraw 绘制业务流程图，导出 PNG
5. 对照 `checklist.md` 自查
6. 确认 UTF-8 编码后提交
