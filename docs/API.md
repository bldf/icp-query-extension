# 查询接口

扩展通过这个接口获取当前网站的 ICP、Whois、DNS 和 GEO 信息。接口地址由用户在扩展选项页的「API 地址」中填写。

## 请求

```
GET <API 地址>?url=<主机名>&version=2&icp=1&whois=1&dns=1
```

### 查询参数

| 参数 | 必填 | 取值 | 说明 |
|---|---|---|---|
| `url` | 是 | 主机名，如 `www.baidu.com` | 当前标签页的 hostname，不带协议、端口和路径，可能包含子域名 |
| `version` | 是 | `2` | 接口版本，固定为 `2` |
| `icp` | 否 | `1` | 出现时返回 `icp` 字段 |
| `whois` | 否 | `1` | 出现时返回 `whois` 字段 |
| `dns` | 否 | `1` | 出现时返回 `dns` 字段（含 GEO） |

- `icp`、`whois`、`dns` 对应选项页的显示开关，关闭时该参数不出现，未请求的模块可以不返回。
- API 地址中原有的查询参数（如 `?token=xxx`）会原样保留，可用于鉴权。
- 请求不带自定义请求头和请求体。

### 示例

```
GET https://api.example.com/icp?token=abc123&url=www.baidu.com&version=2&whois=1&dns=1&icp=1
```

## 响应

### 状态码

| 状态码 | 含义 | 扩展显示 |
|---|---|---|
| `200` | 成功，响应体为 JSON | 查询结果 |
| `429` | 查询次数超限 | 今日查询次数已达上限 |
| 其他 | 服务异常 | 服务器暂时不可用，请稍后重试 |

扩展等待 30 秒，超时显示「服务器响应超时，请稍后重试」。响应体不是合法 JSON 时按服务异常处理。

### 响应头

| 响应头 | 值 |
|---|---|
| `Content-Type` | `application/json` |
| `Access-Control-Allow-Origin` | `*`，或扩展的来源 `chrome-extension://<扩展 ID>` |

### 响应体

一个 JSON 对象。返回数组时只取第一个元素。

```json
{
  "icp": {
    "subject": {
      "name": "北京百度网讯科技有限公司",
      "nature": "企业",
      "license": "京ICP证030173号",
      "updateTime": "2023-05-29 08:20:36"
    },
    "website": {
      "domain": "baidu.com",
      "license": "京ICP证030173号-1"
    }
  },
  "whois": {
    "Domain Status": [
      "clientDeleteProhibited https://icann.org/epp#clientDeleteProhibited",
      "serverTransferProhibited https://icann.org/epp#serverTransferProhibited"
    ],
    "Name Server": ["NS1.BAIDU.COM", "NS2.BAIDU.COM"],
    "Created Date": "1999-10-11T11:05:17Z",
    "Updated Date": "2022-09-01T03:54:43Z",
    "Expiry Date": "2026-10-11T11:05:17Z",
    "Registrar": "MarkMonitor Inc.",
    "Registrant Organization": "Beijing Baidu Netcom Science Technology Co., Ltd.",
    "Registrant Country": "CN"
  },
  "dns": {
    "A": ["180.101.50.242", "180.101.50.188"],
    "AAAA": [],
    "CNAME": ["www.a.shifen.com"],
    "NS": [],
    "GEO": {
      "isp": "电信",
      "area": "中国 江苏 南京"
    }
  }
}
```

## 字段说明

所有字段均可省略，省略的字段显示为 `-`。

### 顶层

| 字段 | 类型 | 说明 |
|---|---|---|
| `msg` | string | 查询失败原因，显示为「查询未完成」错误卡片；其他模块仍会显示 |
| `warn` | string | 提示信息，显示在结果底部 |
| `icp` | object | ICP 备案信息 |
| `whois` | object | Whois 信息 |
| `dns` | object | DNS 解析与 IP 归属信息 |

### `icp`

| 字段 | 类型 | 说明 |
|---|---|---|
| `msg` | string | 无备案信息时的说明，如 `该域名未备案` |
| `subject` | object | 备案主体 |
| `subject.name` | string | 主办单位名称 |
| `subject.nature` | string | 主办单位性质，如 `企业`、`个人`；为 `企业` 时名称链接到天眼查 |
| `subject.license` | string | 主体备案号，如 `京ICP证030173号` |
| `subject.updateTime` | string | 审核通过日期，显示为「备案日期」 |
| `website` | object | 网站备案 |
| `website.domain` | string | 备案域名 |
| `website.license` | string | 网站备案号，如 `京ICP证030173号-1` |

`subject` 和 `website` 同时存在时才显示备案详情，否则显示 `msg`。未备案时返回：

```json
{ "icp": { "msg": "该域名未备案" } }
```

### `whois`

| 字段 | 类型 | 说明 |
|---|---|---|
| `Domain Status` | string[] | 域名状态；每项末尾的 URL 不显示 |
| `Name Server` | string[] | DNS 服务器 |
| `Created Date` | string | 注册日期 |
| `Updated Date` | string | 更新日期 |
| `Expiry Date` | string | 到期日期 |
| `Registrar` | string | 注册商 |
| `Registrant Organization` | string | 注册人组织 |
| `Registrant Country` | string | 注册人国家或地区 |

- 字段名包含空格，大小写需与上表一致。
- 数组字段也可以是以换行或英文逗号分隔的字符串。
- 日期建议使用带时区的 ISO 8601 格式（如 `1999-10-11T11:05:17Z`），扩展按北京时间显示；无法解析的值原样显示。

### `dns`

| 字段 | 类型 | 说明 |
|---|---|---|
| `A` | string[] | A 记录（IPv4） |
| `AAAA` | string[] | AAAA 记录（IPv6） |
| `CNAME` | string[] | CNAME 记录 |
| `NS` | string[] | NS 记录 |
| `GEO` | object | IP 归属信息；省略时不显示该部分 |
| `GEO.area` | string | 地理位置，如 `中国 江苏 南京` |
| `GEO.isp` | string | 运营商，如 `电信` |

数组字段也可以是以换行或英文逗号分隔的字符串。

## 错误响应示例

参数缺失或域名无效时，可以返回 `200` 并在 `msg` 中说明：

```json
{ "msg": "域名格式不正确" }
```

查询次数超限时返回 `429`，响应体可为空。
