# {接口业务名} 接口文档

> 复制本模板后，将 `{}` 占位符替换为实际内容，并删除本提示行。
> 本模板严格按 5 章节组织：接口描述 / 接口路径 / 请求与响应参数 / 错误码 / 业务流程图。

## 1. 接口描述

{一句话定位接口用途}，{1-2 句描述适用业务场景}。

---

## 2. 接口路径

| 项 | 值 |
|----|----|
| Method | `GET` / `POST` |
| Content-Type | `application/json` 或 N/A（GET 请求无 body） |
| 字符编码 | UTF-8 |

### 多环境 URL

| 环境 | URL |
|------|-----|
| qa（测试） | `http://flightadminapi.qa.17usoft.com/gdsaccmng{path}` |
| uat（用户验收） | `http://flightadminapi.uat.17usoft.com/gdsaccmng{path}` |
| pre（预发） | `http://flightadminapi.t.17usoft.com/gdsaccmng{path}` |
| prd（生产） | `http://flightadminapi.17usoft.com/gdsaccmng{path}` |

> 将 `{path}` 替换为实际接口路径，例如 `/officeprintchannel/findByOffice/{officeNo}`。

---

## 3. 请求参数 与 响应参数

### 3.1 请求参数

| 参数名 | 位置 | 类型 | 是否必填 | 说明 |
|--------|------|------|----------|------|
| `{参数名}` | Path / Query / Body | `{类型}` | 是/否 | {含义；枚举值；长度/范围；编码要求} |

#### 请求示例

curl：

```bash
curl -X {METHOD} "{URL}" \
     -H "Content-Type: application/json" \
     -d '{"key":"value"}'
```

Body（POST 时）：

```json
{
  "field1": "value1"
}
```

### 3.2 响应参数

外层统一响应：

| 字段 | 类型 | 必返回 | 说明 |
|------|------|--------|------|
| `success` | `boolean` | 是 | 业务是否成功 |
| `errorCode` | `String` | 失败时返回 | 错误码 |
| `errorMessage` | `String` | 失败时返回 | 错误描述 |
| `data` | `{类型}` | 成功时返回 | 业务数据，详见下表 |

`data` 字段结构：

| 字段 | 类型 | 必返回 | 说明 |
|------|------|--------|------|
| `{字段名}` | `{类型}` | 是/否 | {含义} |

枚举值说明（如有）：

| 字段 | 取值 | 含义 |
|------|------|------|
| `{字段}` | `{value}` | {含义} |

#### 响应示例

成功：

```json
{
  "success": true,
  "errorCode": null,
  "errorMessage": null,
  "data": { ... }
}
```

失败：

```json
{
  "success": false,
  "errorCode": "1000",
  "errorMessage": "参数错误",
  "data": null
}
```

---

## 4. 错误码

业务错误统一通过响应体 `success=false` + `errorCode` 表达，HTTP 状态码均为 `200`。

| errorCode | 含义 | 触发条件 | 调用方应对 |
|-----------|------|---------|------------|
| `0000` | 成功 | 业务正常 | 处理 `data` |
| `1000` | 参数错误 | 入参缺失或非法 | 校验后重试 |
| `2000` | 操作失败 | 业务执行失败 | 查看 `errorMessage` |
| `9999` | 系统异常 | 服务端未预期异常 | 退避重试 |

HTTP 层异常：

| HTTP 状态 | 含义 | 应对方式 |
|-----------|------|---------|
| `404` | 路径错误或服务未部署 | 核对 URL |
| `5xx` | 服务端不可用 | 退避重试 |

---

## 5. 业务流程图

![业务流程图](./images/{doc-name}-flow.png)

> 流程图源文件：`docs/api/excalidraw/{doc-name}-flow.excalidraw`
