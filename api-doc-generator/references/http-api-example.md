# 示例：用户配置查询接口（HTTP）

> 本文件演示按 5 章节标准撰写的完整范例。

## 1. 接口描述

按业务编码查询用户配置项列表。适用于第三方系统在业务流程中实时获取用户的生效配置信息。

---

## 2. 接口路径

| 项 | 值 |
|----|----|
| Method | `GET` |
| Content-Type | N/A |
| 字符编码 | UTF-8 |

### 多环境 URL

| 环境 | URL |
|------|-----|
| qa（测试） | `http://flightadminapi.qa.17usoft.com/gdsaccmng/userconfig/findByBizCode/{bizCode}` |
| uat（用户验收） | `http://flightadminapi.uat.17usoft.com/gdsaccmng/userconfig/findByBizCode/{bizCode}` |
| pre（预发） | `http://flightadminapi.t.17usoft.com/gdsaccmng/userconfig/findByBizCode/{bizCode}` |
| prd（生产） | `http://flightadminapi.17usoft.com/gdsaccmng/userconfig/findByBizCode/{bizCode}` |

---

## 3. 请求参数 与 响应参数

### 3.1 请求参数

| 参数名 | 位置 | 类型 | 是否必填 | 说明 |
|--------|------|------|----------|------|
| `bizCode` | Path | `String` | 是 | 业务编码，长度 1-32，需 URL 编码，区分大小写 |

请求示例：

```bash
curl -X GET "http://flightadminapi.qa.17usoft.com/gdsaccmng/userconfig/findByBizCode/BIZ001"
```

### 3.2 响应参数

外层统一响应：

| 字段 | 类型 | 必返回 | 说明 |
|------|------|--------|------|
| `success` | `boolean` | 是 | 业务是否成功 |
| `errorCode` | `String` | 失败时返回 | 错误码 |
| `errorMessage` | `String` | 失败时返回 | 错误描述 |
| `data` | `Array<Object>` | 成功时返回 | 配置对象数组，无结果时为 `[]` |

`data` 数组元素结构：

| 字段 | 类型 | 必返回 | 说明 |
|------|------|--------|------|
| `id` | `Long` | 是 | 配置记录唯一 ID |
| `bizCode` | `String` | 是 | 业务编码 |
| `channel` | `String` | 是 | 渠道枚举值 |
| `status` | `Integer` | 是 | 状态：`1`=启用，`0`=停用 |

枚举值：

| 字段 | 取值 | 含义 |
|------|------|------|
| `channel` | `ONLINE` | 线上渠道 |
| `channel` | `OFFLINE` | 线下渠道 |
| `status` | `1` | 启用 |
| `status` | `0` | 停用 |

成功响应示例：

```json
{
  "success": true,
  "errorCode": null,
  "errorMessage": null,
  "data": [
    {
      "id": 1001,
      "bizCode": "BIZ001",
      "channel": "ONLINE",
      "status": 1
    }
  ]
}
```

失败响应示例：

```json
{
  "success": false,
  "errorCode": "1000",
  "errorMessage": "bizCode 不能为空",
  "data": null
}
```

---

## 4. 错误码

| errorCode | 含义 | 触发条件 | 调用方应对 |
|-----------|------|---------|------------|
| `0000` | 成功 | 业务正常 | 处理 `data` |
| `1000` | 参数错误 | `bizCode` 缺失或非法 | 校验入参后重试 |
| `2000` | 操作失败 | 业务执行失败 | 查看 `errorMessage` |
| `9999` | 系统异常 | 服务端未预期异常 | 退避重试 |

HTTP 层异常：

| HTTP 状态 | 含义 | 应对方式 |
|-----------|------|---------|
| `404` | 路径错误或服务未部署 | 核对 URL |
| `5xx` | 服务端不可用 | 退避重试 |

---

## 5. 业务流程图

![业务流程图](./images/userconfig-find-by-bizcode-flow.png)

> 流程图源文件：`docs/api/excalidraw/userconfig-find-by-bizcode-flow.excalidraw`
