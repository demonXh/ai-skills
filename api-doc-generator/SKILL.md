---
name: api-doc-generator
description: 项目对外 HTTP 接口文档生成规范 - 为第三方系统调用的 HTTP REST 接口生成精简、标准化的接口文档。只包含接口描述、多环境路径、请求/响应参数、错误码、业务流程图 5 部分内容，不暴露内部实现细节。当用户要求"生成接口文档"、"补充 API 文档"、"为这个接口写文档"、"导出接口说明"时触发。
license: MIT
compatibility: HTTP REST API
---

本 skill 规范项目中所有面向**第三方调用方**的 HTTP 接口文档撰写口径。文档面向外部调用方，**只描述调用契约，不暴露内部实现细节**。

---

## 1. 适用范围

### 1.1 应该生成文档的接口

- Controller 中暴露给外部系统调用的 HTTP REST endpoint
- 跨团队、跨系统通过 HTTP 调用的接口

### 1.2 不需要生成文档的代码

- 内部 Java 客户端封装（`*Client`/`*ClientImpl`）—— 这些是同公司内部 SDK，对外只需暴露其底层 HTTP 接口
- Service 接口、DAO、工具类、Controller 中仅本前端使用的 endpoint
- 内部 DTO、VO、Form、枚举类的独立文档

---

## 2. 文档存放约定

| 约定项 | 规则 |
|--------|------|
| 存放目录 | `docs/api/` |
| 文件名 | 按业务命名，与对外接口的业务名一致，例如 `office-printer-query.md`。**不使用内部 Java 类名作为文件名** |
| 编码 | **必须 UTF-8（无 BOM）**，禁止 GBK |
| 一个文件一个对外接口（或同一业务下多个相关 endpoint） |

---

## 3. 文档章节结构（强制，仅 5 部分）

```
1. 接口描述
2. 接口路径（区分 qa / uat / 预发 / 生产，含请求方式 GET/POST）
3. 请求参数 与 响应参数
4. 错误码
5. 业务流程图（excalidraw 绘制）
```

**禁止包含**：
- Java 代码示例、`@Resource` 注入、SDK 用法
- 内部类名（`PrinterVO`、`ResultMsg`、`OfficePrinterClient` 等）
- 数据库表名、SQL、Redis Key、缓存实现
- "特殊行为"、"FAQ"、"快速开始"、"调用示例"等额外章节
- 内部异常类、堆栈、日志格式

**允许出现**：
- curl 示例（用于演示真实 HTTP 请求结构）
- JSON 请求/响应示例
- 业务字段含义、枚举值表格

---

## 4. 各章节撰写规范

### 4.1 接口描述

- 一句话定位 + 1-2 句适用场景
- 不超过 5 行
- 用业务语言描述，不提"Controller"、"Service"、"Client"等内部概念

### 4.2 接口路径

必须用表格列出 4 个环境的完整 URL：

| 环境 | URL |
|------|-----|
| qa（测试） | `http://{host}.qa{domain}{path}` |
| uat（用户验收） | `http://{host}.uat{domain}{path}` |
| pre（预发，对应内部 `stage`） | `http://{host}.t{domain}{path}` |
| prd（生产） | `http://{host}{domain}{path}` |

外加单独说明：

| 项 | 值 |
|----|----|
| Method | `GET` / `POST` |
| Content-Type | `application/json` 或 N/A |
| 字符编码 | UTF-8 |

### 4.3 请求参数 与 响应参数

**请求参数表格**（Path / Query / Body 分开列）：

| 参数名 | 位置 | 类型 | 是否必填 | 说明 |
|--------|------|------|----------|------|
| `xxx` | Path / Query / Body | `String` / `Integer` / `Long` / `Boolean` / `Array` / `Object` | 是/否 | 含义；枚举值；长度/范围；编码要求 |

**响应参数表格**（外层 + data 内层分开列）：

| 字段 | 类型 | 必返回 | 说明 |
|------|------|--------|------|
| `success` | `boolean` | 是 | 业务是否成功 |
| `data` | ... | 成功时返回 | 业务数据 |
| ... | | | |

枚举值用独立表格补充：

| 枚举字段 | 取值 | 含义 |
|----------|------|------|
| `channel` | `D` | 国内 |
| `channel` | `I` | 国际 |

最后必须给出**JSON 响应示例**。

### 4.4 错误码

| errorCode | 含义 | 触发条件 | 调用方应对 |
|-----------|------|---------|------------|
| `0000` | 成功 | 业务正常 | 处理 data |
| ... | ... | ... | ... |

HTTP 层异常单独列：

| HTTP 状态 | 含义 | 应对方式 |
|-----------|------|---------|

### 4.5 业务流程图

**强制使用 excalidraw 绘制**，并在文档中嵌入图片或 excalidraw 链接。

流程图内容要求：
- 描述**业务调用流程**（调用方 → 接口 → 关键判断 → 返回）
- 节点用矩形/菱形/箭头组合
- **不暴露内部模块**（数据库、缓存、Redis 等），最多体现"服务内部处理"一个黑盒
- 文档中插入图片引用：`![业务流程图](./images/<doc-name>-flow.png)`
- excalidraw 源文件保存到 `docs/api/excalidraw/<doc-name>-flow.excalidraw`

---

## 5. 工作流（强制）

```
1. 读 Controller 源码 → 抽 endpoint、Method、参数、返回类型
2. 读响应 VO/DTO → 抽字段、类型、枚举（仅取对外字段含义）
3. 读环境配置 → 拼出 qa/uat/pre/prd 4 个完整 URL
4. 写 1-4 章节（接口描述、路径、参数、错误码）
5. 调用 excalidraw MCP 绘制业务流程图
6. 导出 PNG 到 docs/api/images/，源文件保存到 docs/api/excalidraw/
7. 在文档中插入图片引用
8. 对照 references/checklist.md 自查
9. 写入 docs/api/<business-name>.md（UTF-8）
```

---

## 6. 项目环境 URL 映射（参考）

本项目对外 HTTP 接口的 host 模板为：

```
http://flightadminapi{envSuffix}.17usoft.com/gdsaccmng
```

`envSuffix` 取值（与 `UrlGenerator` 一致）：

| 环境 | envSuffix | 完整 host |
|------|-----------|----------|
| qa | `.qa` | `http://flightadminapi.qa.17usoft.com/gdsaccmng` |
| uat | `.uat` | `http://flightadminapi.uat.17usoft.com/gdsaccmng` |
| 预发（stage） | `.t` | `http://flightadminapi.t.17usoft.com/gdsaccmng` |
| 生产（product） | `（空）` | `http://flightadminapi.17usoft.com/gdsaccmng` |

撰写接口路径章节时，把 `{path}` 替换为该 endpoint 的实际 path 即可。

---

## 7. 通用响应格式

项目所有 HTTP 接口统一使用如下响应壳：

```json
{
  "success": true,
  "errorCode": null,
  "errorMessage": null,
  "data": { ... }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `success` | `boolean` | 业务是否成功 |
| `errorCode` | `String` | 错误码，成功时为 `null` |
| `errorMessage` | `String` | 错误描述，成功时为 `null` |
| `data` | `Object`/`Array`/`基本类型` | 业务数据 |

通用错误码（所有接口共享）：

| errorCode | 含义 |
|-----------|------|
| `0000` | 成功 |
| `1000` | 参数错误 |
| `2000` | 操作失败 |
| `9999` | 系统异常 |

撰写文档时，错误码章节只需列出本接口**额外**触发的业务错误码（如有），并引用通用错误码即可。

---

## 8. 参考资源

| 文件 | 用途 |
|------|------|
| `references/doc-template.md` | 5 章节标准模板 |
| `references/http-api-example.md` | HTTP 接口完整范例（含 4 环境 URL 与流程图占位） |
| `references/checklist.md` | 文档完成度自查清单 |
| `references/README.md` | 参考目录索引 |
