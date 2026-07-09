---
name: api-doc-generator
description: "生成 API 文档并写入 17u Wiki。为指定 HTTP 接口或代码方法生成基于源码的接口文档，保存到 17u Wiki 页面。触发词：生成接口文档、API 文档、导出接口说明、写 Wiki 文档。"
version: 2.0.0
author: Demon
license: MIT
---

# 生成 API 文档并写入 17u Wiki

为一个指定接口或代码方法生成基于源码的接口文档，然后保存到指定的 17u Wiki 页面。

## 必需输入

如果缺少任意一项，先询问用户再继续：

- **API 目标**：HTTP 路径（如 `/marketing/queryPromotion`）、完整路由、或代码符号（如 `ApiController.queryPromotion`）
- **Wiki 目标**：`https://wiki.17u.cn/wiki?fid=...` 或 `https://toca.17u.cn/wiki?fid=...` URL

## 工作流程

### 第一步：确认 Wiki 认证状态

执行任何 Wiki 操作前，先确认认证状态：

```bash
node ~/.wiki-17u/scripts/wiki.mjs doc-status
```

如果认证缺失或已过期，运行初始化脚本：

```bash
node ~/.wiki-17u/scripts/setup.mjs
```

该脚本会优先尝试连接当前已打开的 Chrome 调试实例；失败后再自动打开浏览器登录；最后才走手动 Cookie 输入。

**安全规则**：不要打印 token 或 `~/.wiki-17u/config.json` 的内容。

### 第二步：在代码中定位 API 目标

优先使用 `rg` 搜索：

```bash
rg -n "<path-or-method-name>" .
```

阅读以下源码，确保文档基于真实实现：

- Controller、路由常量
- 请求 DTO、响应 DTO
- Service 方法
- 参数组装类
- 下游 client
- 相关测试/规格说明

**规则**：源码可用时，不要只凭字段名猜测。

### 第三步：生成 Markdown 文档

文档必须包含以下章节：

1. **标题**
2. **接口概述**（一句话定位 + 1-2 句适用场景，不超过 5 行）
3. **基础信息表**（Method、Content-Type、字符编码、多环境 URL）
4. **入参说明表**（字段、位置、类型、必填、说明）
5. **嵌套入参说明表**（如有，用点号/数组下标记法，如 `searchParams.ext.pageId`）
6. **返参说明表**（字段、类型、说明）
7. **场景/分支行为表**（当接口存在多个模式时补充）
8. **入参示例**（语法正确的 JSON）
9. **返参示例**（语法正确的 JSON，匹配 DTO 结构）
10. **异常/边界行为表**
11. **错误码表**
12. **代码位置**（Controller 类名、方法名、文件路径）

### 第四步：保存 Markdown 草稿

将草稿保存到仓库外部临时目录：

```bash
# Windows
/tmp/api-doc.md 或 $TEMP/api-doc.md

# 也可以保存到项目 docs/api/ 目录
```

### 第五步：写入 Wiki

```bash
node ~/.wiki-17u/scripts/write-wiki-markdown.mjs --url '<wiki-url>' --input /tmp/api-doc.md
```

脚本支持：标题、段落、Markdown 表格、fenced code block。

它会优先从环境变量 `WIKI_17U_TOKEN` / `WIKI_17U_COOKIE` 读取认证信息，其次读取 `~/.wiki-17u/config.json`。

### 第六步：验证写入结果

写入后重新拉取页面验证：

```bash
node ~/.wiki-17u/scripts/wiki.mjs fetch-doc '<wiki-url>' --outputDir=/tmp/wiki-verify --meta --no-images
```

确认：
- 表格仍然是表格
- 示例仍然是代码块
- 章节结构完整

## 文档生成规则

1. **入参和返参字段必须使用表格**。入参表包含：字段、类型、必填、说明；返参表包含：字段、类型、说明。
2. **嵌套字段使用点号/数组下标记法**，例如 `searchParams.ext.pageId` 和 `data.promotions[].actCode`。
3. **条件必填字段要明确标注**，例如"条件必填"，并说明具体条件。
4. **记录代码真实实现的行为**，包括返回 null、默认值、字段优先级和下游调用。
5. **示例必须是语法正确的 JSON**，并且要匹配文档说明的 DTO 结构。
6. **外部 DTO 处理**：如果导入的 DTO 来自外部依赖且本地没有源码，只记录本地代码/测试能证明的字段，或说明该对象遵循外部 DTO 契约。
7. **安全红线**：不要写入密钥、真实 token、完整 Cookie、密码，或配置/缓存文件中的内部凭据。

## 通用响应格式

项目 HTTP 接口统一响应壳：

```json
{
  "success": true,
  "errorCode": null,
  "errorMessage": null,
  "data": { ... }
}
```

通用错误码：

| errorCode | 含义 | 触发条件 | 调用方应对 |
|-----------|------|---------|------------|
| `0000` | 成功 | 业务正常 | 处理 data |
| `1000` | 参数错误 | 入参缺失或非法 | 校验后重试 |
| `2000` | 操作失败 | 业务执行失败 | 查看 errorMessage |
| `9999` | 系统异常 | 服务端未预期异常 | 退避重试 |

文档错误码章节只需列出本接口**额外**触发的业务错误码，并引用通用错误码。

## 项目环境 URL 映射

host 模板：`http://flightadminapi{envSuffix}.17usoft.com/gdsaccmng`

| 环境 | envSuffix | 完整 host |
|------|-----------|----------|
| qa | `.qa` | `http://flightadminapi.qa.17usoft.com/gdsaccmng` |
| uat | `.uat` | `http://flightadminapi.uat.17usoft.com/gdsaccmng` |
| 预发 | `.t` | `http://flightadminapi.t.17usoft.com/gdsaccmng` |
| 生产 | （空） | `http://flightadminapi.17usoft.com/gdsaccmng` |

## 禁止事项

- 不写入密钥、真实 token、完整 Cookie、密码
- 不暴露数据库表名、SQL、Redis Key、缓存实现
- 不包含 Java 代码示例（`@Resource`、`import`、SDK 调用）
- 不出现内部类名（`*VO` / `*DTO` / `*Client` / `*Service`）在对外文档中
- 不包含"快速开始"、"FAQ"等额外章节

## 脚本依赖

| 脚本 | 用途 |
|------|------|
| `~/.wiki-17u/scripts/wiki.mjs` | Wiki 认证状态检查、文档拉取 |
| `~/.wiki-17u/scripts/setup.mjs` | Wiki 认证初始化（Chrome 调试 / 浏览器登录 / 手动 Cookie） |
| `~/.wiki-17u/scripts/write-wiki-markdown.mjs` | 将 Markdown 写入 Wiki 页面 |

## 参考资源

| 文件 | 用途 |
|------|------|
| `references/doc-template.md` | 文档模板 |
| `references/http-api-example.md` | HTTP 接口完整范例 |
| `references/checklist.md | 文档完成度自查清单 |
