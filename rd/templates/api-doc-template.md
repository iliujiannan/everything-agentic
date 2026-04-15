# 联通云7（WoCloud7）OpenAPI 对接分享

> 文档版本：v2  
> 生成时间：2026-04-08  
> 核心文件：`WoCloud7RestApi.java`、`WoCloud7Helper.java`、`WoCloud7SignatureUtil.java`

---

## 目录

1. [背景与架构概述](#1-背景与架构概述)
2. [认证机制详解](#2-认证机制详解)
3. [公共结构说明](#3-公共结构说明)
4. [接口详情](#4-接口详情)
   - 4.1 云区域查询
   - 4.2 可用区查询
   - 4.3 VPC 管理
   - 4.4 子网管理
   - 4.5 安全组管理
   - 4.6 安全组规则管理
   - 4.7 镜像查询
   - 4.8 规格查询
   - 4.9 云硬盘管理
   - 4.10 云主机（VM）管理
   - 4.11 裸金属（BM）管理
5. [分页查询机制](#5-分页查询机制)
6. [接口调用依赖关系](#6-接口调用依赖关系)
7. [注意事项与已知问题](#7-注意事项与已知问题)

---

## 1. 背景与架构概述

本服务通过 **Spring Cloud OpenFeign** 调用联通云7平台的 OpenAPI，完成云资源的统一纳管。

```
本平台
  └─ WoCloud7Helper（封装请求/分页/错误处理）
       └─ WoCloud7RestApi（Feign Client，对应联通云7 REST API）
            └─ 联通云7 OpenAPI Gateway（https://<host>:<port>/）
```

**关键设计原则：**
- 所有请求在发出前先通过 `WoCloud7Helper.generateHeader()` 生成携带签名的请求头
- Feign 接口第一个参数固定为 `URI restEndpoint`，支持运行时动态切换 Base URL（多云商多资源池场景）
- 响应统一用 `WoCloud7ResponseEntryWrapper<T>` 包裹，业务层统一判断 `wrapper.isSuccess()`

---

## 2. 认证机制详解

### 2.1 请求头字段

每个请求均需在 HTTP Header 中携带以下字段，由 `WoCloud7Helper.generateHeader()` 自动生成：

| Header 字段      | 类型     | 固定值 / 说明                         |
|----------------|--------|-----------------------------------|
| `signedHeader` | String | 固定值 `"1"`，表示请求头参与加签              |
| `algorithm`    | String | 固定值 `"HmacSHA256"`                |
| `requestTime`  | String | 当前毫秒时间戳（UTC+8），**有效期 1 分钟**      |
| `accessKey`    | String | 平台分配的 AccessKey ID               |
| `sign`         | String | HmacSHA256 签名结果（小写十六进制字符串）       |

### 2.2 签名算法步骤

```
Step 1: 构建加签参数 Map
        = 所有请求参数（Query/Body 中的非 null 字段）
        + 所有请求头字段（除 "sign" 本身）

Step 2: 按 key 自然排序，拼接为：
        key1="value1"&key2="value2"&...
        （value 使用 FastJSON 序列化）

Step 3: HMAC-SHA256(秘钥=AccessKeySecret, 数据=上述字符串)

Step 4: 字节数组 → 小写十六进制字符串 → 填入 "sign" 请求头
```

**注意事项：**
- 时间戳有效期为 1 分钟，服务器时钟偏差过大会导致签名失败
- POST 接口 Body 内容参与加签，**加签用的 paramMap 必须与实际发送的 Body 完全一致**，特别注意 null 值处理

---

## 3. 公共结构说明

### 3.1 标准响应基类

所有接口响应均继承 `CommonWoCloud7ResponseEntry`：

| 字段名  | 类型     | 说明                              |
|------|--------|---------------------------------|
| code | int    | `200` 成功，其他值失败                  |
| msg  | String | 失败时的错误描述，成功时通常为 null 或 "success" |

### 3.2 分页响应基类

列表类接口额外包含以下分页字段（`CommonWoCloud7PageResponseEntry`）：

| 字段名        | 类型      | 说明    |
|------------|---------|-------|
| total      | Long    | 总记录数  |
| pageNumber | Integer | 当前页码  |
| pages      | Integer | 总页数   |
| pageSize   | Integer | 每页数量  |

### 3.3 旧版响应结构（仅规格接口使用）

`CommonOldWoCloud7ResponseEntry`：

| 字段名     | 类型     | 说明           |
|---------|--------|--------------|
| code    | int    | `200` 成功     |
| message | String | 操作描述         |
| result  | Object | 业务数据（泛型）     |

### 3.4 通用分页请求参数

大多数列表接口均支持（`CommonWoCloud7PageRequestEntry`）：

| 参数名        | 位置    | 类型     | 必须 | 说明               |
|------------|-------|--------|----|------------------|
| cloudId    | Query | String | 否  | 云商 ID             |
| regionId   | Query | String | 否  | 资源池 ID            |
| pageNumber | Query | String | 否  | 页码，从 1 开始，默认 1   |
| pageSize   | Query | String | 否  | 每页数量，默认 10       |

---

## 4. 接口详情

> **Base URL**：`https://<host>:<port>/`（通过 AccountEntry.baseUrl 动态注入）  
> 所有接口均需携带认证请求头，见第 2 章

---

### 4.1 云区域查询

**`GET /cmdb/v1/regions`**

**请求参数：**

| 参数名        | 位置    | 类型     | 必须 | 说明    |
|------------|-------|--------|----|-------|
| cloudId    | Query | String | 否  | 云商 ID |
| regionName | Query | String | 否  | 资源池名称 |
| regionCode | Query | String | 否  | 资源池编码 |

**响应参数：**

| 字段名                | 类型            | 说明       |
|--------------------|---------------|----------|
| code               | int           | 状态码      |
| msg                | String        | 消息       |
| regions            | List\<Object\> | 区域列表     |
| regions[].id       | String        | 区域 ID    |
| regions[].regionName | String      | 区域名称     |
| regions[].regionCode | String      | 区域编码     |
| regions[].regionVersion | String   | 版本       |
| regions[].regionModel | String     | 资源池类型    |
| regions[].area     | String        | 地理区域     |
| regions[].country  | String        | 国家       |
| regions[].provinceName | String    | 省份名称     |
| regions[].eparchyName | String     | 地市名称     |
| regions[].detailAddress | String  | 详细地址     |
| regions[].cloudId  | String        | 所属云商 ID  |
| regions[].cloudCode | String       | 云商编码     |
| regions[].cloudName | String       | 云商名称     |
| regions[].cloudType | String       | 云商类型     |
| regions[].tenantId | String        | 租户 ID    |
| regions[].status   | String        | 状态       |
| regions[].createTime | String      | 创建时间     |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "regions": [
    {
      "id": "1001",
      "regionName": "华北一区",
      "regionCode": "cn-north-1",
      "regionVersion": "v2",
      "regionModel": "public",
      "area": "华北",
      "country": "中国",
      "provinceName": "北京",
      "eparchyName": "北京",
      "detailAddress": "北京市海淀区",
      "cloudId": "2329616615270678528",
      "cloudCode": "wocloud7",
      "cloudName": "联通云7",
      "cloudType": "public",
      "tenantId": null,
      "status": "active",
      "createTime": "2024-01-01 00:00:00"
    }
  ]
}
```

---

### 4.2 可用区查询

**`GET /cmdb/v1/available-zones`**

**请求参数：**

| 参数名      | 位置    | 类型     | 必须 | 说明    |
|----------|-------|--------|----|-------|
| cloudId  | Query | String | 否  | 云商 ID |
| regionId | Query | String | 否  | 资源池 ID |

**响应参数：**

| 字段名          | 类型            | 说明        |
|--------------|---------------|-----------|
| code         | int           | 状态码       |
| msg          | String        | 消息        |
| azs          | List\<Object\> | 可用区列表     |
| azs[].id     | String        | 可用区 ID    |
| azs[].uuid   | String        | 可用区 UUID  |
| azs[].azName | String        | 可用区名称     |
| azs[].azCode | String        | 可用区编码     |
| azs[].regionId | String      | 所属资源池 ID  |
| azs[].regionName | String    | 所属资源池名称   |
| azs[].cloudName | String     | 云商名称      |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "azs": [
    {
      "id": "2001",
      "uuid": "az-uuid-001",
      "azName": "华北一区-AZ1",
      "azCode": "cn-north-1a",
      "regionId": "1001",
      "regionName": "华北一区",
      "cloudName": "联通云7"
    }
  ]
}
```

---

### 4.3 VPC 管理

#### 4.3.1 查询 VPC 列表

**`GET /resource/v2/vpcs`**

支持分页，自动翻页使用 `requestAllPage()`。

**请求参数：**

| 参数名        | 位置    | 类型     | 必须 | 说明    |
|------------|-------|--------|----|-------|
| cloudId    | Query | String | 否  | 云商 ID |
| regionId   | Query | String | 否  | 资源池 ID |
| instanceName | Query | String | 否 | VPC 名称筛选 |
| pageNumber | Query | String | 否  | 页码    |
| pageSize   | Query | String | 否  | 每页数量  |

**响应参数：**

| 字段名                  | 类型            | 说明        |
|----------------------|---------------|-----------|
| code                 | int           | 状态码       |
| msg                  | String        | 消息        |
| total                | Long          | 总记录数      |
| pageNumber           | Integer       | 当前页码      |
| pages                | Integer       | 总页数       |
| pageSize             | Integer       | 每页数量      |
| vpcs                 | List\<Object\> | VPC 列表    |
| vpcs[].instanceId    | String        | VPC 实例 ID |
| vpcs[].instanceName  | String        | VPC 名称    |
| vpcs[].uuid          | String        | 底层 UUID   |
| vpcs[].cidr          | String        | 网段 CIDR   |
| vpcs[].status        | String        | 状态        |
| vpcs[].optStatus     | String        | 操作状态      |
| vpcs[].subnetNumber  | Integer       | 子网数量      |
| vpcs[].serverNumber  | Integer       | 云主机数量     |
| vpcs[].routerNumber  | Integer       | 路由数量      |
| vpcs[].routerId      | String        | 路由 ID     |
| vpcs[].loadBalanceNumber | Integer   | 负载均衡数量    |
| vpcs[].regionId      | String        | 资源池 ID    |
| vpcs[].regionName    | String        | 资源池名称     |
| vpcs[].regionCode    | String        | 资源池编码     |
| vpcs[].cloudId       | String        | 云商 ID     |
| vpcs[].cloudName     | String        | 云商名称      |
| vpcs[].azId          | String        | 可用区 ID    |
| vpcs[].azName        | String        | 可用区名称     |
| vpcs[].projectId     | String        | 项目 ID     |
| vpcs[].projectName   | String        | 项目名称      |
| vpcs[].createTime    | String        | 创建时间      |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "total": 5,
  "pageNumber": 1,
  "pages": 1,
  "pageSize": 50,
  "vpcs": [
    {
      "instanceId": "vpc-abc123",
      "instanceName": "my-vpc",
      "uuid": "vpc-uuid-001",
      "cidr": "192.168.0.0/16",
      "status": "active",
      "optStatus": null,
      "subnetNumber": 2,
      "serverNumber": 5,
      "routerNumber": 1,
      "routerId": "router-001",
      "loadBalanceNumber": 0,
      "regionId": "1001",
      "regionName": "华北一区",
      "regionCode": "cn-north-1",
      "cloudId": "2329616615270678528",
      "cloudCode": "wocloud7",
      "cloudName": "联通云7",
      "azId": "2001",
      "azName": "华北一区-AZ1",
      "azCode": "cn-north-1a",
      "projectId": "3001",
      "projectName": "项目A",
      "createTime": "2024-01-01 10:00:00"
    }
  ]
}
```

---

#### 4.3.2 创建 VPC

**`POST /resource/v2/vpcs`**

**请求参数：**

| 参数名          | 位置   | 类型     | 必须 | 说明       |
|--------------|------|--------|----|----------|
| cloudCode    | Body | String | 是  | 云商编码     |
| cloudId      | Body | String | 否  | 云商 ID    |
| cloudName    | Body | String | 否  | 云商名称     |
| regionId     | Body | String | 是  | 资源池 ID   |
| regionCode   | Body | String | 是  | 资源池编码    |
| regionName   | Body | String | 是  | 资源池名称    |
| azId         | Body | String | 是  | 可用区 ID   |
| azCode       | Body | String | 是  | 可用区编码    |
| azName       | Body | String | 否  | 可用区名称    |
| instanceName | Body | String | 是  | VPC 名称   |
| cidr         | Body | String | 是  | 网段 CIDR  |
| tenantId     | Body | String | 是  | 租户 ID    |
| masterTenantId | Body | String | 是 | 主租户 ID  |
| projectId    | Body | String | 否  | 项目 ID    |

**响应参数：**

| 字段名              | 类型     | 说明        |
|------------------|--------|-----------|
| code             | int    | 状态码       |
| msg              | String | 消息        |
| data             | Object | 创建结果      |
| data.instanceId  | String | 新建 VPC ID |
| data.instanceName | String | 新建 VPC 名称 |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "instanceId": "vpc-new123",
    "instanceName": "my-new-vpc"
  }
}
```

---

#### 4.3.3 删除 VPC

**`DELETE /resource/v2/vpcs/{instanceId}`**

**请求参数：**

| 参数名        | 位置   | 类型     | 必须 | 说明        |
|------------|------|--------|----|-----------|
| instanceId | Path | String | 是  | VPC 实例 ID |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

### 4.4 子网管理

#### 4.4.1 查询子网列表

**`GET /resource/v2/subnets`**

支持分页，自动翻页使用 `requestAllPage()`。

> **已知问题**：联通云7该接口存在分页 bug，框架通过"返回数量 ≠ pageSize 即终止翻页"规避。

**请求参数：**

| 参数名        | 位置    | 类型     | 必须 | 说明    |
|------------|-------|--------|----|-------|
| cloudId    | Query | String | 否  | 云商 ID |
| regionId   | Query | String | 否  | 资源池 ID |
| instanceName | Query | String | 否 | 子网名称筛选 |
| pageNumber | Query | String | 否  | 页码    |
| pageSize   | Query | String | 否  | 每页数量  |

**响应参数：**

| 字段名                    | 类型            | 说明        |
|------------------------|---------------|-----------|
| code                   | int           | 状态码       |
| msg                    | String        | 消息        |
| total                  | Long          | 总记录数      |
| pageNumber             | Integer       | 当前页码      |
| pages                  | Integer       | 总页数       |
| pageSize               | Integer       | 每页数量      |
| subnets                | List\<Object\> | 子网列表      |
| subnets[].instanceId   | String        | 子网实例 ID   |
| subnets[].instanceName | String        | 子网名称      |
| subnets[].uuid         | String        | 底层 UUID   |
| subnets[].cidr         | String        | 子网 CIDR   |
| subnets[].gateway      | String        | 网关地址      |
| subnets[].dns          | String        | DNS 地址    |
| subnets[].enableDhcp   | Integer       | DHCP：1启用，0禁用 |
| subnets[].networkId    | String        | 所属网络 ID   |
| subnets[].routerId     | String        | 路由 ID     |
| subnets[].status       | String        | 状态        |
| subnets[].regionId     | String        | 资源池 ID    |
| subnets[].regionName   | String        | 资源池名称     |
| subnets[].cloudId      | String        | 云商 ID     |
| subnets[].cloudName    | String        | 云商名称      |
| subnets[].azId         | String        | 可用区 ID    |
| subnets[].azName       | String        | 可用区名称     |
| subnets[].createTime   | String        | 创建时间      |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "total": 3,
  "pageNumber": 1,
  "pages": 1,
  "pageSize": 50,
  "subnets": [
    {
      "instanceId": "subnet-abc123",
      "instanceName": "my-subnet",
      "uuid": "subnet-uuid-001",
      "cidr": "192.168.1.0/24",
      "gateway": "192.168.1.1",
      "dns": "8.8.8.8",
      "enableDhcp": 1,
      "networkId": "net-001",
      "routerId": "router-001",
      "status": "active",
      "optStatus": null,
      "regionId": "1001",
      "regionName": "华北一区",
      "regionCode": "cn-north-1",
      "cloudId": "2329616615270678528",
      "cloudName": "联通云7",
      "azId": "2001",
      "azName": "华北一区-AZ1",
      "createTime": "2024-01-02 10:00:00"
    }
  ]
}
```

---

#### 4.4.2 创建子网

**`POST /resource/v2/subnets`**

**请求参数：**

| 参数名            | 位置   | 类型      | 必须 | 说明                  |
|----------------|------|---------|-----|---------------------|
| vpcId          | Body | String  | 是   | 所属 VPC ID           |
| instanceName   | Body | String  | 是   | 子网名称                |
| cidr           | Body | String  | 是   | 子网 CIDR             |
| gateway        | Body | String  | 是   | 网关（直通型不传）           |
| dns            | Body | String  | 是   | DNS 地址              |
| ipVersion      | Body | Integer | 是   | IP 版本：`4`（IPv4）/ `6`（IPv6） |
| regionId       | Body | Long    | 是   | 资源池 ID              |
| tenantId       | Body | String  | 是   | 租户 ID               |
| masterTenantId | Body | String  | 是   | 主租户 ID              |
| azId           | Body | Long    | 否   | 可用区 ID              |
| cloudId        | Body | String  | 否   | 云商 ID               |
| cloudCode      | Body | String  | 否   | 云商编码                |
| description    | Body | String  | 否   | 子网描述                |
| segmentId      | Body | String  | 否   | 用户网络段 ID（直通型必填）     |

**响应参数：**

| 字段名               | 类型     | 说明        |
|-------------------|--------|-----------|
| code              | int    | 状态码       |
| msg               | String | 消息        |
| data              | Object | 创建结果      |
| data.instanceId   | String | 新建子网 ID   |
| data.instanceName | String | 新建子网名称    |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "instanceId": "subnet-new456",
    "instanceName": "my-new-subnet"
  }
}
```

---

#### 4.4.3 删除子网

**`DELETE /v2/vpcs/{vpcId}/subnets/{subnetId}`**

> 注意：路径前缀为 `/v2`（不含 `/resource`），与创建接口不同。

**请求参数：**

| 参数名      | 位置   | 类型     | 必须 | 说明    |
|----------|------|--------|----|-------|
| vpcId    | Path | String | 是  | VPC ID |
| subnetId | Path | String | 是  | 子网 ID |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

### 4.5 安全组管理

#### 4.5.1 查询安全组列表

**`GET /resource/v1/security-groups`**

支持分页，自动翻页使用 `requestAllPage()`，List 字段为 `securityGroups`。

**请求参数：**

| 参数名        | 位置    | 类型     | 必须 | 说明    |
|------------|-------|--------|----|-------|
| cloudId    | Query | String | 否  | 云商 ID |
| regionId   | Query | String | 否  | 资源池 ID |
| instanceName | Query | String | 否 | 安全组名称筛选 |
| pageNumber | Query | String | 否  | 页码    |
| pageSize   | Query | String | 否  | 每页数量  |

**响应参数：**

| 字段名                               | 类型            | 说明        |
|------------------------------------|---------------|-----------|
| code                               | int           | 状态码       |
| msg                                | String        | 消息        |
| total                              | Long          | 总记录数      |
| pageNumber                         | Integer       | 当前页码      |
| pages                              | Integer       | 总页数       |
| pageSize                           | Integer       | 每页数量      |
| securityGroups                     | List\<Object\> | 安全组列表     |
| securityGroups[].instanceId        | String        | 安全组实例 ID  |
| securityGroups[].instanceName      | String        | 安全组名称     |
| securityGroups[].securityGroupExtName | String     | 安全组扩展名称   |
| securityGroups[].relateNumber      | Integer       | 关联实例数量    |
| securityGroups[].rulesNumber       | Integer       | 规则数量      |
| securityGroups[].status            | String        | 状态        |
| securityGroups[].optStatus         | String        | 操作状态      |
| securityGroups[].regionId          | String        | 资源池 ID    |
| securityGroups[].regionName        | String        | 资源池名称     |
| securityGroups[].cloudId           | String        | 云商 ID     |
| securityGroups[].cloudName         | String        | 云商名称      |
| securityGroups[].createTime        | String        | 创建时间      |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "total": 10,
  "pageNumber": 1,
  "pages": 1,
  "pageSize": 50,
  "securityGroups": [
    {
      "instanceId": "sg-abc123",
      "instanceName": "my-security-group",
      "securityGroupExtName": "my-security-group",
      "relateNumber": 3,
      "rulesNumber": 5,
      "status": "active",
      "optStatus": null,
      "regionId": "1001",
      "regionName": "华北一区",
      "cloudId": "2329616615270678528",
      "cloudName": "联通云7",
      "createTime": "2024-01-03 10:00:00"
    }
  ]
}
```

---

#### 4.5.2 创建安全组

**`POST /resource/v1/security-group`**

**请求参数：**

| 参数名               | 位置   | 类型     | 必须 | 说明               |
|-------------------|------|--------|----|------------------|
| cloudId           | Body | Long   | 是  | 云商 ID            |
| regionId          | Body | Long   | 是  | 资源池 ID           |
| securityGroupName | Body | String | 是  | 安全组名称            |
| projectId         | Body | Long   | 否  | 项目 ID（2024-08-13 确认非必填）|
| tenantId          | Body | Long   | 否  | 租户 ID            |

**响应参数：**

| 字段名          | 类型     | 说明       |
|--------------|--------|----------|
| code         | int    | 状态码      |
| msg          | String | 消息       |
| instanceId   | String | 新建安全组 ID |
| instanceName | String | 新建安全组名称  |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "instanceId": "sg-new789",
  "instanceName": "my-new-sg"
}
```

---

#### 4.5.3 删除安全组

**`DELETE /resource/v1/security-group/{securityGroupId}`**

**请求参数：**

| 参数名             | 位置   | 类型     | 必须 | 说明     |
|-----------------|------|--------|----|--------|
| securityGroupId | Path | String | 是  | 安全组 ID |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.5.4 更新安全组名称

**`PUT /resource/v1/security-group/change-group-name`**

**请求参数：**

| 参数名               | 位置   | 类型     | 必须 | 说明      |
|-------------------|------|--------|----|---------|
| securityGroupId   | Body | Long   | 是  | 安全组 ID  |
| securityGroupName | Body | String | 是  | 新安全组名称  |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

### 4.6 安全组规则管理

#### 4.6.1 查询安全组规则列表

**`GET /resource/v1/security-group-rules`**

支持分页，自动翻页使用 `requestAllPage()`，List 字段为 `groupRules`。

**请求参数：**

| 参数名             | 位置    | 类型     | 必须 | 说明         |
|-----------------|-------|--------|----|------------|
| securityGroupId | Query | String | 否  | 过滤指定安全组 ID |
| cloudId         | Query | String | 否  | 云商 ID       |
| regionId        | Query | String | 否  | 资源池 ID      |
| pageNumber      | Query | String | 否  | 页码          |
| pageSize        | Query | String | 否  | 每页数量        |

**响应参数：**

| 字段名                               | 类型            | 说明                           |
|------------------------------------|---------------|------------------------------|
| code                               | int           | 状态码                          |
| msg                                | String        | 消息                           |
| total                              | Long          | 总记录数                         |
| pageNumber                         | Integer       | 当前页码                         |
| pages                              | Integer       | 总页数                          |
| pageSize                           | Integer       | 每页数量                         |
| groupRules                         | List\<Object\> | 规则列表                         |
| groupRules[].groupRuleId           | String        | 规则 ID（用于删除）                  |
| groupRules[].securityGroupInstanceId | String      | 所属安全组实例 ID                   |
| groupRules[].direction             | String        | 方向：`ingress`/`egress`        |
| groupRules[].protocol              | String        | 协议：`tcp`/`udp`/`icmp`/`any`  |
| groupRules[].portRangeMin          | Integer       | 起始端口                         |
| groupRules[].portRangeMax          | Integer       | 终止端口                         |
| groupRules[].remoteIpPrefix        | String        | 授权 IP 段（CIDR 格式）             |
| groupRules[].ipVersion             | String        | IP 版本：`4`/`6`                |
| groupRules[].action                | String        | 动作：`allow`/`deny`            |
| groupRules[].priority              | Integer       | 优先级，1~100，越小越高               |
| groupRules[].status                | Integer       | 状态                           |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "total": 4,
  "pageNumber": 1,
  "pages": 1,
  "pageSize": 50,
  "groupRules": [
    {
      "groupRuleId": "rule-001",
      "securityGroupInstanceId": "sg-abc123",
      "direction": "ingress",
      "protocol": "tcp",
      "portRangeMin": 22,
      "portRangeMax": 22,
      "remoteIpPrefix": "0.0.0.0/0",
      "ipVersion": "4",
      "action": "allow",
      "priority": 1,
      "status": 1
    },
    {
      "groupRuleId": "rule-002",
      "securityGroupInstanceId": "sg-abc123",
      "direction": "ingress",
      "protocol": "tcp",
      "portRangeMin": 443,
      "portRangeMax": 443,
      "remoteIpPrefix": "0.0.0.0/0",
      "ipVersion": "4",
      "action": "allow",
      "priority": 10,
      "status": 1
    }
  ]
}
```

---

#### 4.6.2 创建安全组规则

**`POST /resource/v1/security-group-rule`**

> **已知问题**：联通云7该接口不返回规则 ID（2024-08-27 确认），创建后需调用查询接口获取 `groupRuleId`。

**请求参数：**

| 参数名                          | 位置   | 类型      | 必须 | 说明                                         |
|------------------------------|------|---------|----|--------------------------------------------|
| securityGroupId              | Body | Long    | 否  | 安全组 ID                                     |
| groupRuleList                | Body | List    | 否  | 规则集合（支持批量创建）                               |
| groupRuleList[].direction    | Body | String  | 否  | `ingress`（入方向）/ `egress`（出方向）             |
| groupRuleList[].ipVersion    | Body | Integer | 否  | `4`（IPv4）/ `6`（IPv6）                      |
| groupRuleList[].protocol     | Body | String  | 否  | `tcp` / `udp` / `icmp` / `any`             |
| groupRuleList[].remoteIpPrefix | Body | String | 否  | CIDR 格式，如 `0.0.0.0/0`                     |
| groupRuleList[].portRangeMin | Body | Integer | 否  | 起始端口（protocol=any 时可不传）                    |
| groupRuleList[].portRangeMax | Body | Integer | 否  | 终止端口（protocol=any 时可不传）                    |
| groupRuleList[].action       | Body | String  | 否  | `allow`（允许，默认）/ `deny`（拒绝）                |
| groupRuleList[].priority     | Body | String  | 否  | 优先级 `"1"`~`"100"`，越小越高，默认 `"1"`           |

**响应参数：**

| 字段名  | 类型     | 说明                    |
|------|--------|-----------------------|
| code | int    | 状态码                   |
| msg  | String | 消息（不含新建规则 ID，这是已知问题） |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.6.3 删除安全组规则

**`DELETE /resource/v1/security-group-rule/{groupRuleId}`**

**请求参数：**

| 参数名         | 位置   | 类型     | 必须 | 说明    |
|-------------|------|--------|----|-------|
| groupRuleId | Path | String | 是  | 规则 ID |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

### 4.7 镜像查询

#### 查询镜像列表

**`GET /cmdb/v2/images`**

**请求参数：**

| 参数名             | 位置    | 类型     | 必须 | 说明                                           |
|-----------------|-------|--------|----|----------------------------------------------|
| cloudId         | Query | String | 否  | 云商 ID                                         |
| regionId        | Query | String | 否  | 资源池 ID                                        |
| name            | Query | String | 否  | 镜像名称筛选                                        |
| imageType       | Query | String | 否  | `common`（公共镜像）/ `whole`（整机镜像）/ `private`（私有镜像）|
| operationStatus | Query | String | 否  | `up`（上架）/ `down`（下架）                        |
| status          | Query | String | 否  | 状态                                             |
| bootMode        | Query | String | 否  | 启动方式                                           |
| pageNumber      | Query | String | 否  | 页码                                             |
| pageSize        | Query | String | 否  | 每页数量                                           |

**响应参数：**

| 字段名                      | 类型            | 说明               |
|--------------------------|---------------|------------------|
| code                     | int           | 状态码              |
| msg                      | String        | 消息               |
| images                   | List\<Object\> | 镜像列表             |
| images[].uuid            | String        | 镜像 UUID（创建VM时使用）|
| images[].name            | String        | 镜像名称             |
| images[].imageOs         | String        | 操作系统             |
| images[].imageOsType     | String        | 系统类型：`linux`/`windows` |
| images[].imageType       | String        | 镜像类型             |
| images[].imageBit        | Integer       | 位数：`32`/`64`    |
| images[].frameworkType   | String        | 架构：`x86_64`/`aarch64` |
| images[].volumeType      | String        | 磁盘类型             |
| images[].minRam          | Integer       | 最小内存（MB）         |
| images[].size            | String        | 镜像大小             |
| images[].description     | String        | 描述               |
| images[].operationStatus | String        | 上下架状态            |
| images[].status          | String        | 状态               |
| images[].resourceType    | String        | 适用资源类型           |
| images[].cloudName       | String        | 云商名称             |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "images": [
    {
      "uuid": "img-uuid-001",
      "name": "CentOS-7.9-x86_64",
      "imageOs": "CentOS",
      "imageOsType": "linux",
      "imageType": "common",
      "imageVersion": null,
      "imageBit": 64,
      "frameworkType": "x86_64",
      "volumeType": "SSD",
      "minRam": 2048,
      "size": "40GB",
      "description": "CentOS 7.9 标准镜像",
      "operationStatus": "up",
      "status": "active",
      "resourceType": "VM",
      "cloudName": "联通云7"
    }
  ]
}
```

---

### 4.8 规格查询

#### 查询规格列表

**`POST /product/product/productquery/flavors`**

> 使用**旧版分页协议**（`currPage`/`pageSize` 均为 Integer 类型），自动翻页使用 `requestAllPageOld()`。  
> 该接口为 POST，Body 内容**全部参与加签**，含 null 值的字段也不能遗漏。

**请求参数：**

| 参数名      | 位置   | 类型      | 必须 | 说明                         |
|----------|------|---------|-----|----------------------------|
| resType  | Body | String  | 是   | 资源类型，如 `"VM"`、`"BMS"`     |
| cloudId  | Body | String  | 是   | 云商 ID                      |
| regionId | Body | String  | 是   | 资源池 ID                     |
| currPage | Body | Integer | 否   | 当前页，默认 1                   |
| pageSize | Body | Integer | 否   | 每页数量，默认 10                 |

**响应参数：**

| 字段名                         | 类型      | 说明          |
|-----------------------------|---------|-------------|
| code                        | int     | 状态码         |
| message                     | String  | 消息          |
| result                      | Object  | 分页结果        |
| result.totalCount           | Integer | 规格总数        |
| result.pageSize             | Integer | 分页大小        |
| result.totalPage            | Integer | 总页数         |
| result.currPage             | Integer | 当前页         |
| result.list                 | List    | 规格列表        |
| result.list[].productId     | String  | 规格 ID（创建VM时使用）|
| result.list[].productName   | String  | 规格名称        |
| result.list[].productDesc   | String  | 规格描述        |
| result.list[].productDetail | String  | 规格详情（JSON 字符串）|
| result.list[].billType      | String  | 计费类型        |
| result.list[].billUnit      | String  | 计费单位        |
| result.list[].productArchitect | String | 架构         |
| result.list[].productMode   | String  | 产品模式        |
| result.list[].productClass  | String  | 规格类别        |
| result.list[].productPrice  | Integer | 产品价格        |
| result.list[].resourceId    | String  | 第三方 ID      |
| result.list[].resType       | String  | 资源类型        |
| result.list[].instanceFamily | String | 规格族         |

**响应示例：**

```json
{
  "code": 200,
  "message": "操作成功",
  "result": {
    "totalCount": 120,
    "pageSize": 50,
    "totalPage": 3,
    "currPage": 1,
    "list": [
      {
        "productId": "flavor-001",
        "productName": "2核4G通用型",
        "productDesc": "2 vCPU / 4GB 内存",
        "productDetail": "[{\"prtyCode\":\"cpu\",\"prtyValue\":\"2\"},{\"prtyCode\":\"memory\",\"prtyValue\":\"4096\"}]",
        "billType": "postpaid",
        "billUnit": "hour",
        "productArchitect": "x86_64",
        "productMode": "shared",
        "productClass": "general",
        "productPrice": 100,
        "resourceId": "ecs.c6.large",
        "resType": "VM",
        "instanceFamily": "c6"
      }
    ]
  }
}
```

---

### 4.9 云硬盘管理

#### 4.9.1 查询云硬盘列表

**`GET /resource/v1/volumes`**

支持分页，自动翻页使用 `requestAllPage()`，List 字段为 `volumes`。

**请求参数：**

| 参数名        | 位置    | 类型     | 必须 | 说明    |
|------------|-------|--------|----|-------|
| cloudId    | Query | String | 否  | 云商 ID |
| regionId   | Query | String | 否  | 资源池 ID |
| instanceName | Query | String | 否 | 磁盘名称筛选 |
| status     | Query | String | 否  | 状态筛选  |
| pageNumber | Query | String | 否  | 页码    |
| pageSize   | Query | String | 否  | 每页数量  |

**响应参数：**

| 字段名                         | 类型            | 说明            |
|-----------------------------|---------------|---------------|
| code                        | int           | 状态码           |
| msg                         | String        | 消息            |
| total                       | Long          | 总记录数          |
| pageNumber                  | Integer       | 当前页码          |
| pages                       | Integer       | 总页数           |
| pageSize                    | Integer       | 每页数量          |
| volumes                     | List\<Object\> | 磁盘列表          |
| volumes[].instanceId        | String        | 磁盘实例 ID       |
| volumes[].instanceName      | String        | 磁盘名称          |
| volumes[].uuid              | String        | 底层 UUID       |
| volumes[].capacity          | Integer       | 磁盘容量（GB）      |
| volumes[].volumeType        | String        | 磁盘类型          |
| volumes[].volumeProperty    | String        | 磁盘属性（系统盘/数据盘）|
| volumes[].volumeSpecialType | String        | 特殊类型          |
| volumes[].diskMode          | String        | 磁盘模式          |
| volumes[].flavorId          | String        | 规格 ID         |
| volumes[].status            | String        | 状态：`available`/`in-use`/`error` |
| volumes[].optStatus         | String        | 操作状态          |
| volumes[].isEnable          | Integer       | 是否可用          |
| volumes[].vmId              | String        | 已挂载的主机 ID（未挂载为 null）|
| volumes[].vmName            | String        | 已挂载的主机名称      |
| volumes[].vmUuid            | String        | 已挂载主机 UUID    |
| volumes[].regionId          | String        | 资源池 ID        |
| volumes[].regionName        | String        | 资源池名称         |
| volumes[].cloudId           | String        | 云商 ID         |
| volumes[].cloudName         | String        | 云商名称          |
| volumes[].azId              | String        | 可用区 ID        |
| volumes[].azName            | String        | 可用区名称         |
| volumes[].projectId         | String        | 项目 ID         |
| volumes[].createTime        | String        | 创建时间          |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "total": 20,
  "pageNumber": 1,
  "pages": 1,
  "pageSize": 50,
  "volumes": [
    {
      "instanceId": "vol-abc123",
      "instanceName": "my-data-disk",
      "uuid": "vol-uuid-001",
      "capacity": 100,
      "volumeType": "SSD",
      "volumeProperty": "data",
      "volumeSpecialType": "normal",
      "volumeSpecialTypeName": "普通云硬盘",
      "diskMode": "VBD",
      "flavorId": "disk-flavor-001",
      "status": "available",
      "optStatus": null,
      "isEnable": 1,
      "vmId": null,
      "vmName": null,
      "vmUuid": null,
      "regionId": "1001",
      "regionName": "华北一区",
      "cloudId": "2329616615270678528",
      "cloudName": "联通云7",
      "azId": "2001",
      "azName": "华北一区-AZ1",
      "projectId": "3001",
      "createTime": "2024-02-01 10:00:00"
    }
  ]
}
```

---

#### 4.9.2 云硬盘详情

**`GET /resource/v1//volume/{volume_id}`**

> 路径中存在双斜杠 `//`，为联通云7 API 原始路径，非笔误。

**请求参数：**

| 参数名       | 位置   | 类型     | 必须 | 说明    |
|-----------|------|--------|----|-------|
| volume_id | Path | String | 是  | 磁盘 ID |

**响应参数：**

与 4.9.1 查询云硬盘列表响应参数相同（返回 `volumes` 数组，通常只含一条记录）。

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "total": 1,
  "volumes": [
    {
      "instanceId": "vol-abc123",
      "instanceName": "my-data-disk",
      "capacity": 100,
      "volumeType": "SSD",
      "status": "in-use",
      "vmId": "vm-server-001",
      "vmName": "my-vm-001"
    }
  ]
}
```

---

#### 4.9.3 创建云硬盘

**`POST /resource/v1/volumes`**

**请求参数：**

| 参数名            | 位置   | 类型      | 必须 | 说明         |
|----------------|------|---------|-----|------------|
| cloudId        | Body | String  | 是   | 云商 ID      |
| regionId       | Body | String  | 是   | 资源池 ID     |
| azId           | Body | String  | 是   | 可用区 ID     |
| projectId      | Body | String  | 是   | 项目 ID      |
| flavorId       | Body | String  | 是   | 磁盘规格 ID    |
| capacity       | Body | Integer | 是   | 磁盘大小（GB）   |
| instanceName   | Body | String  | 是   | 磁盘名称       |
| tenantId       | Body | String  | 否   | 租户 ID      |
| masterTenantId | Body | String  | 否   | 主租户 ID     |
| serviceType    | Body | Integer | 否   | 服务类型       |
| isShare        | Body | Boolean | 否   | 是否共享盘      |
| isIscsi        | Body | Integer | 否   | 是否 iSCSI 模式 |

**响应参数：**

| 字段名          | 类型     | 说明        |
|--------------|--------|-----------|
| code         | int    | 状态码       |
| msg          | String | 消息        |
| instanceId   | String | 新建磁盘 ID   |
| instanceName | String | 新建磁盘名称    |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "instanceId": "vol-new456",
  "instanceName": "my-new-disk"
}
```

---

#### 4.9.4 删除云硬盘

**`DELETE /resource/v1/volume/delete/{volume_id}`**

**请求参数：**

| 参数名        | 位置   | 类型     | 必须 | 说明              |
|------------|------|--------|----|-----------------|
| volume_id  | Path | String | 是  | 磁盘 ID（路径参数）     |
| instanceId | Body | String | 是  | 磁盘 ID（Body 中同传）|

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.9.5 挂载云硬盘

**`PUT /resource/v1//volume/mount`**

> 路径中存在双斜杠 `//`，为联通云7 API 原始路径。

**请求参数：**

| 参数名       | 位置   | 类型           | 必须 | 说明             |
|-----------|------|--------------|-----|----------------|
| volumeId  | Body | String       | 是   | 磁盘 ID          |
| serverIds | Body | List\<String\> | 是   | 要挂载的云主机 ID 列表  |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.9.6 卸载云硬盘

**`PUT /resource/v1//volume/unmount`**

**请求参数：**

| 参数名       | 位置   | 类型           | 必须 | 说明             |
|-----------|------|--------------|-----|----------------|
| volumeId  | Body | String       | 是   | 磁盘 ID          |
| serverIds | Body | List\<String\> | 是   | 要卸载的云主机 ID 列表  |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.9.7 扩容云硬盘

**`PUT /resource/v1/volume/expansion`**

**请求参数：**

| 参数名        | 位置   | 类型      | 必须 | 说明           |
|------------|------|---------|-----|--------------|
| instanceId | Body | String  | 是   | 磁盘 ID        |
| capacity   | Body | Integer | 是   | 扩容后的容量（GB）   |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

### 4.10 云主机（VM）管理

#### 4.10.1 查询 VM 列表

**`GET /resource/v1/servers`**

支持分页，自动翻页使用 `requestAllPage()`，List 字段为 `servers`。

**请求参数：**

| 参数名          | 位置    | 类型     | 必须 | 说明                     |
|--------------|-------|--------|----|------------------------|
| cloudId      | Query | String | 否  | 云商 ID                   |
| regionId     | Query | String | 否  | 资源池 ID                  |
| status       | Query | String | 否  | 状态，如 `running`/`stopped` |
| instanceId   | Query | String | 否  | 精确匹配实例 ID               |
| instanceName | Query | String | 否  | 模糊匹配实例名称                |
| ipv4Address  | Query | String | 否  | 按 IP 过滤                 |
| azIds        | Query | Long[] | 否  | 可用区 ID 数组               |
| pageNumber   | Query | String | 否  | 页码                      |
| pageSize     | Query | String | 否  | 每页数量                    |

**响应参数：**

| 字段名                      | 类型            | 说明             |
|--------------------------|---------------|----------------|
| code                     | int           | 状态码            |
| msg                      | String        | 消息             |
| total                    | Long          | 总记录数           |
| pageNumber               | Integer       | 当前页码           |
| pages                    | Integer       | 总页数            |
| pageSize                 | Integer       | 每页数量           |
| servers                  | List\<Object\> | VM 列表          |
| servers[].instanceId     | String        | VM 实例 ID       |
| servers[].instanceName   | String        | VM 名称          |
| servers[].uuid           | String        | 底层 UUID        |
| servers[].ipv4Address    | String        | 内网 IP          |
| servers[].publicIp       | String        | 公网 IP（未绑定为 null）|
| servers[].isBandIp       | Integer       | 是否绑定公网：1是，0否  |
| servers[].cpu            | Integer       | CPU 核数         |
| servers[].memory         | Long          | 内存（MB）         |
| servers[].osType         | String        | 操作系统类型         |
| servers[].osName         | String        | 操作系统名称         |
| servers[].flavorId       | Long          | 规格 ID          |
| servers[].subnetInstanceId | String      | 子网 ID          |
| servers[].status         | String        | 状态             |
| servers[].optStatus      | String        | 操作状态           |
| servers[].regionId       | String        | 资源池 ID         |
| servers[].regionName     | String        | 资源池名称          |
| servers[].cloudId        | String        | 云商 ID          |
| servers[].cloudName      | String        | 云商名称           |
| servers[].azId           | String        | 可用区 ID         |
| servers[].azName         | String        | 可用区名称          |
| servers[].projectId      | String        | 项目 ID          |
| servers[].projectName    | String        | 项目名称           |
| servers[].createTime     | String        | 创建时间           |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "total": 30,
  "pageNumber": 1,
  "pages": 1,
  "pageSize": 50,
  "servers": [
    {
      "instanceId": "vm-server-001",
      "instanceName": "my-vm-001",
      "uuid": "vm-uuid-001",
      "ipv4Address": "192.168.1.100",
      "publicIp": null,
      "isBandIp": 0,
      "cpu": 4,
      "memory": 8192,
      "osType": "linux",
      "osName": "CentOS 7.9",
      "flavorId": 10001,
      "subnetInstanceId": "subnet-abc123",
      "status": "running",
      "optStatus": null,
      "regionId": "1001",
      "regionName": "华北一区",
      "cloudId": "2329616615270678528",
      "cloudName": "联通云7",
      "azId": "2001",
      "azName": "华北一区-AZ1",
      "projectId": "3001",
      "projectName": "项目A",
      "createTime": "2024-03-01 10:00:00"
    }
  ]
}
```

---

#### 4.10.2 VM 详情

**`GET /resource/v1/servers/{server_id}`**

**请求参数：**

| 参数名       | 位置   | 类型     | 必须 | 说明     |
|-----------|------|--------|----|--------|
| server_id | Path | String | 是  | VM 实例 ID |

**响应参数：**

与 4.10.1 查询 VM 列表响应参数相同（返回 `servers` 数组，通常只含一条记录）。

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "servers": [
    {
      "instanceId": "vm-server-001",
      "instanceName": "my-vm-001",
      "ipv4Address": "192.168.1.100",
      "cpu": 4,
      "memory": 8192,
      "osName": "CentOS 7.9",
      "status": "running"
    }
  ]
}
```

---

#### 4.10.3 创建 VM

**`POST /resource/v1/servers`**

**请求参数：**

| 参数名                          | 位置   | 类型      | 必须 | 说明                                         |
|------------------------------|------|---------|----|--------------------------------------------|
| cloudId                      | Body | String  | 是  | 云商 ID                                      |
| regionId                     | Body | String  | 是  | 资源池 ID                                     |
| azId                         | Body | String  | 是  | 可用区 ID                                     |
| projectId                    | Body | String  | 否  | 项目 ID                                      |
| tenantId                     | Body | String  | 否  | 租户 ID                                      |
| instanceName                 | Body | String  | 否  | VM 名称                                      |
| password                     | Body | String  | 否  | 登录密码                                       |
| subPassword                  | Body | String  | 否  | 确认密码                                       |
| count                        | Body | Integer | 否  | 创建数量，默认 1                                  |
| flavor.flavorUuid            | Body | String  | 是  | 规格产品 ID（来自规格查询的 `productId`）              |
| image.imageUuid              | Body | String  | 是  | 镜像 UUID（来自镜像查询的 `uuid`）                   |
| volume.volumeSys.capacity    | Body | Integer | 否  | 系统盘容量（GB）                                  |
| volume.volumeSys.productId   | Body | String  | 是  | 系统盘产品 ID                                   |
| network.vpcDefault           | Body | Integer | 是  | VPC 是否默认：`1`默认 / `0`选择                    |
| network.vpcId                | Body | String  | 否  | VPC ID（vpcDefault=0 时必填）                   |
| network.subnetDefault        | Body | Integer | 是  | 子网是否默认：`1`默认 / `0`选择                      |
| network.subnetId             | Body | String  | 否  | 子网 ID（subnetDefault=0 时必填）                 |
| security.defaultSecurity     | Body | Integer | 是  | 安全组是否默认：`1`默认 / `0`选择                     |
| security.securityGroupId     | Body | String  | 否  | 安全组 ID（defaultSecurity=0 时必填）              |
| floatIp.accessIpMode         | Body | Integer | 是  | 公网 IP 模式：`0`立即创建 / `1`使用已有 / `2`暂不创建    |

**响应参数：**

| 字段名                       | 类型            | 说明        |
|---------------------------|---------------|-----------|
| code                      | int           | 状态码       |
| msg                       | String        | 消息        |
| instances                 | List\<Object\> | 创建的 VM 列表 |
| instances[].instanceId    | String        | VM 实例 ID  |
| instances[].instanceName  | String        | VM 名称     |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "instances": [
    {
      "instanceId": "vm-new-001",
      "instanceName": "my-new-vm"
    }
  ]
}
```

---

#### 4.10.4 删除 VM

**`DELETE /resource/v1/servers/{server_id}`**

**请求参数：**

| 参数名       | 位置   | 类型     | 必须 | 说明     |
|-----------|------|--------|----|--------|
| server_id | Path | String | 是  | VM 实例 ID |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.10.5 VM 关机

**`GET /resource/v1/servers/shut-down/{server_id}`**

**请求参数：**

| 参数名       | 位置   | 类型     | 必须 | 说明     |
|-----------|------|--------|----|--------|
| server_id | Path | String | 是  | VM 实例 ID |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.10.6 VM 开机

**`GET /resource/v1/servers/power-on/{server_id}`**

**请求参数：**

| 参数名       | 位置   | 类型     | 必须 | 说明     |
|-----------|------|--------|----|--------|
| server_id | Path | String | 是  | VM 实例 ID |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.10.7 VM 重启

**`GET /resource/v1/servers/restart/{server_id}`**

**请求参数：**

| 参数名       | 位置   | 类型     | 必须 | 说明     |
|-----------|------|--------|----|--------|
| server_id | Path | String | 是  | VM 实例 ID |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.10.8 修改 VM 名称

**`PUT /resource/v1/servers/update-instance-name`**

**请求参数：**

| 参数名          | 位置   | 类型     | 必须 | 说明     |
|--------------|------|--------|----|--------|
| serverId     | Body | Long   | 是  | VM 实例 ID |
| instanceName | Body | String | 是  | 新名称    |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.10.9 修改 VM 规格

**`PUT /resource/v1/servers/update-flavor`**

**请求参数：**

| 参数名        | 位置   | 类型     | 必须 | 说明           |
|------------|------|--------|----|--------------|
| cloudId    | Body | String | 否  | 云商 ID        |
| regionId   | Body | String | 否  | 资源池 ID       |
| serverId   | Body | String | 是  | VM 实例 ID     |
| projectId  | Body | String | 否  | 项目 ID        |
| tenantId   | Body | String | 否  | 租户 ID        |
| flavorUuid | Body | String | 是  | 新规格 UUID     |
| chargeType | Body | String | 否  | 计费类型         |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.10.10 重置 VM 密码

**`PUT /resource/v1/servers/update-password`**

**请求参数：**

| 参数名         | 位置   | 类型     | 必须 | 说明    |
|-------------|------|--------|----|-------|
| serverId    | Body | Long   | 是  | VM 实例 ID |
| password    | Body | String | 是  | 新密码   |
| subPassword | Body | String | 是  | 确认密码  |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.10.11 获取 VM VNC 地址

**`GET /resource/v1/servers/remote-console/{server_id}`**

**请求参数：**

| 参数名       | 位置   | 类型     | 必须 | 说明     |
|-----------|------|--------|----|--------|
| server_id | Path | String | 是  | VM 实例 ID |

**响应参数：**

| 字段名                   | 类型     | 说明              |
|-----------------------|--------|-----------------|
| code                  | int    | 状态码             |
| msg                   | String | 消息              |
| remote_console        | Object | VNC 控制台信息（下划线命名）|
| remote_console.url    | String | VNC 访问地址        |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "remote_console": {
    "url": "https://vnc.wocloud7.example.com/vnc?token=xxxxxxxx"
  }
}
```

---

### 4.11 裸金属（BM）管理

#### 4.11.1 查询裸金属列表

**`GET /baremet/v1/baremetals`**

支持分页，自动翻页使用 `requestAllPage()`，List 字段为 `servers`。

**请求参数：**

| 参数名        | 位置    | 类型     | 必须 | 说明    |
|------------|-------|--------|----|-------|
| cloudId    | Query | String | 否  | 云商 ID |
| regionId   | Query | String | 否  | 资源池 ID |
| instanceName | Query | String | 否 | 裸金属名称筛选 |
| status     | Query | String | 否  | 状态筛选  |
| pageNumber | Query | String | 否  | 页码    |
| pageSize   | Query | String | 否  | 每页数量  |

**响应参数：**

| 字段名                    | 类型            | 说明        |
|------------------------|---------------|-----------|
| code                   | int           | 状态码       |
| msg                    | String        | 消息        |
| total                  | Long          | 总记录数      |
| pageNumber             | Integer       | 当前页码      |
| pages                  | Integer       | 总页数       |
| pageSize               | Integer       | 每页数量      |
| servers                | List\<Object\> | 裸金属列表     |
| servers[].instanceId   | String        | 裸金属实例 ID  |
| servers[].instanceName | String        | 裸金属名称     |
| servers[].id           | String        | 内部 ID     |
| servers[].uuid         | String        | 底层 UUID   |
| servers[].ipv4Address  | String        | 内网 IP     |
| servers[].publicIp     | String        | 公网 IP     |
| servers[].cpu          | Integer       | CPU 核数    |
| servers[].memory       | Integer       | 内存（MB）    |
| servers[].osType       | String        | 操作系统类型    |
| servers[].osName       | String        | 操作系统名称    |
| servers[].flavorId     | String        | 规格 ID     |
| servers[].flavorType   | String        | 规格类型      |
| servers[].hostId       | String        | 物理主机 ID   |
| servers[].hostName     | String        | 物理主机名称    |
| servers[].status       | String        | 状态        |
| servers[].optStatus    | String        | 操作状态      |
| servers[].externalDiskInfo | String    | 磁盘信息      |
| servers[].regionId     | String        | 资源池 ID    |
| servers[].regionName   | String        | 资源池名称     |
| servers[].cloudId      | String        | 云商 ID     |
| servers[].cloudName    | String        | 云商名称      |
| servers[].azId         | String        | 可用区 ID    |
| servers[].azName       | String        | 可用区名称     |
| servers[].projectId    | String        | 项目 ID     |
| servers[].projectName  | String        | 项目名称      |
| servers[].createTime   | String        | 创建时间      |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "total": 5,
  "pageNumber": 1,
  "pages": 1,
  "pageSize": 50,
  "servers": [
    {
      "instanceId": "bm-server-001",
      "instanceName": "my-bm-001",
      "id": "bm-id-001",
      "uuid": "bm-uuid-001",
      "ipv4Address": "192.168.10.1",
      "publicIp": null,
      "cpu": 48,
      "memory": 262144,
      "osType": "linux",
      "osName": "CentOS 7.9",
      "flavorId": "bm-flavor-001",
      "flavorType": "BMS",
      "hostId": "host-001",
      "hostName": "physical-host-001",
      "status": "running",
      "optStatus": "active",
      "externalDiskInfo": null,
      "regionId": "1001",
      "regionName": "华北一区",
      "cloudId": "2329616615270678528",
      "cloudName": "联通云7",
      "azId": "2001",
      "azName": "华北一区-AZ1",
      "projectId": "3001",
      "projectName": "项目A",
      "createTime": "2024-04-01 10:00:00"
    }
  ]
}
```

---

#### 4.11.2 创建裸金属

**`POST /baremet/v1/baremetals`**

**请求参数：**

| 参数名                   | 位置   | 类型      | 必须 | 说明                                         |
|-----------------------|------|---------|----|--------------------------------------------|
| cloudId               | Body | Long    | 是  | 云商 ID                                      |
| cloudName             | Body | String  | 是  | 云商名称                                       |
| regionId              | Body | Long    | 是  | 资源池 ID                                     |
| azId                  | Body | Long    | 是  | 可用区 ID（需为 ironic 开头的 az）                   |
| projectId             | Body | Long    | 是  | 项目 ID                                      |
| tenantId              | Body | Long    | 否  | 租户 ID                                      |
| instanceName          | Body | String  | 是  | 裸金属名称                                      |
| count                 | Body | Integer | 是  | 创建数量                                       |
| password              | Body | String  | 是  | 登录密码                                       |
| subPassword           | Body | String  | 是  | 确认密码                                       |
| bondMode              | Body | String  | 是  | 网卡绑定模式：`bond1`（推荐主备）/ `bond4`（推荐聚合）       |
| flavor.flavorUuid     | Body | String  | 是  | 规格 UUID                                    |
| image.imageUuid       | Body | String  | 是  | 镜像 UUID                                    |
| image.type            | Body | String  | 是  | 镜像类型：`linux` / `windows`                  |
| network.vpcDefault    | Body | Integer | 是  | VPC 是否默认：`1`默认 / `0`选择                    |
| network.vpcId         | Body | Long    | 否  | VPC ID，传子网的 `networkId`（vpcDefault=0 时必填） |
| network.subnetDefault | Body | Integer | 是  | 子网是否默认：`1`默认 / `0`选择                      |
| network.subnetId      | Body | Long    | 否  | 子网 ID（subnetDefault=0 时必填）                 |
| floatIp.accessIpMode  | Body | Integer | 是  | 公网 IP 模式：`0`立即创建 / `1`使用已有 / `2`暂不创建    |

**响应参数：**

| 字段名                       | 类型            | 说明         |
|---------------------------|---------------|------------|
| code                      | int           | 状态码        |
| msg                       | String        | 消息         |
| instances                 | List\<Object\> | 创建的裸金属列表   |
| instances[].instanceId    | String        | 裸金属实例 ID   |
| instances[].instanceName  | String        | 裸金属名称      |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "instances": [
    {
      "instanceId": "bm-new-001",
      "instanceName": "my-new-bm"
    }
  ]
}
```

---

#### 4.11.3 删除裸金属

**`DELETE /baremet/v1/baremetals/{server_id}`**

**请求参数：**

| 参数名       | 位置   | 类型     | 必须 | 说明      |
|-----------|------|--------|----|---------|
| server_id | Path | String | 是  | 裸金属实例 ID |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.11.4 裸金属开机

**`GET /baremet/v1/baremetals/power-on/{server_id}`**

**请求参数：**

| 参数名       | 位置   | 类型     | 必须 | 说明      |
|-----------|------|--------|----|---------|
| server_id | Path | String | 是  | 裸金属实例 ID |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.11.5 裸金属关机

**`GET /baremet/v1/baremetals/shut-down/{server_id}`**

**请求参数：**

| 参数名       | 位置   | 类型     | 必须 | 说明      |
|-----------|------|--------|----|---------|
| server_id | Path | String | 是  | 裸金属实例 ID |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.11.6 裸金属重启

**`GET /baremet/v1/baremetals/restart/{server_id}`**

**请求参数：**

| 参数名       | 位置   | 类型     | 必须 | 说明      |
|-----------|------|--------|----|---------|
| server_id | Path | String | 是  | 裸金属实例 ID |

**响应参数：**

| 字段名  | 类型     | 说明   |
|------|--------|------|
| code | int    | 状态码  |
| msg  | String | 消息   |

**响应示例：**

```json
{
  "code": 200,
  "msg": "success"
}
```

---

#### 4.11.7 裸金属 VNC

**`GET /baremet/v1/baremetals/remote-console/{server_id}`**

**请求参数：**

| 参数名       | 位置   | 类型     | 必须 | 说明      |
|-----------|------|--------|----|---------|
| server_id | Path | String | 是  | 裸金属实例 ID |

**响应参数：**

| 字段名         | 类型     | 说明                  |
|-------------|--------|---------------------|
| code        | int    | 状态码                 |
| msg         | String | 消息                  |
| vncPassword | String | VNC 密码              |
| url         | String | VNC 地址（因网络限制通常不可直接访问）|

**响应示例：**

```json
{
  "code": 200,
  "msg": "success",
  "vncPassword": "random-vnc-password",
  "url": "https://vnc.wocloud7.example.com/vnc?token=xxxxxxxx"
}
```

---

## 5. 分页查询机制

### 5.1 新版分页（大多数列表接口）

| 请求参数       | 类型     | 说明             |
|------------|--------|----------------|
| pageNumber | String | 页码，从 1 开始      |
| pageSize   | String | 每页数量，建议 50     |

**自动翻页终止条件（`requestAllPage()`，满足任一即停止）：**

1. 接口返回 `code != 200` → 抛出 `CloudNetException`
2. 本页返回数量 `< pageSize` → 已到最后一页
3. 本页返回为空

> 条件 2 是为了规避联通云7 subnets 接口分页 bug：即使数据已无更多，接口仍循环返回同一批 pageSize 条数据。

### 5.2 旧版分页（规格接口专用）

| 请求参数     | 类型      | 说明         |
|----------|---------|------------|
| currPage | Integer | 当前页，从 1 开始 |
| pageSize | Integer | 每页数量       |

**自动翻页终止条件（`requestAllPageOld()`）：**
本页返回数量 `< pageSize` 或返回为空。

---

## 6. 接口调用依赖关系

```
创建 VM / BM 之前必须准备：
  queryRegions()               → regionId
    └─ queryZones()            → azId
         ├─ queryVpcs()        → vpcId（或 createVpc）
         │    └─ querySubnets()  → subnetId（或 createSubnet）
         ├─ queryNsgs()        → securityGroupId（或 createNsg）
         │    └─ queryNsgRules() → groupRuleId（或 createNsgRule）
         ├─ queryImages()      → imageUuid
         └─ queryFlavors()     → productId（flavorUuid）

安全组规则操作：
  createNsg() / queryNsgs()    → instanceId（作为 securityGroupId 传入）
    └─ createNsgRule(securityGroupId)
    └─ queryNsgRules(securityGroupId) → groupRuleId
         └─ deleteNsgRule(groupRuleId)

删除子网（需要双路径参数）：
  queryVpcs()   → vpcId（instanceId）
  querySubnets() → subnetId（instanceId）
    └─ deleteSubnet(vpcId, subnetId)

磁盘挂载操作：
  createDisk() / queryDisks()  → instanceId（volumeId）
  createVm() / queryVms()      → instanceId（serverId）
    └─ attachDisk(volumeId, serverIds)
    └─ detachDisk(volumeId, serverIds)
    └─ expansionDisk(instanceId, capacity)
```

---

## 7. 注意事项与已知问题

| # | 问题描述                                                    | 影响接口                                             | 处理方式                                               |
|---|----------------------------------------------------------|--------------------------------------------------|----------------------------------------------------|
| 1 | 子网列表接口分页 bug，最后一页仍返回 pageSize 条                          | `querySubnets`                                   | 框架通过"返回数量 ≠ pageSize 即停止"规避                        |
| 2 | 创建安全组规则响应**不含规则 ID**（2024-08-27 确认）                     | `createNsgRule`                                  | 创建后通过 `queryNsgRules` 重新查询，用 `groupRuleId` 字段标识    |
| 3 | 裸金属 VNC 的 `url` 因网络限制无法直接访问                              | `bmVnc`                                          | 仅使用 `vncPassword` 字段，`url` 废弃不用                     |
| 4 | 磁盘详情 / 挂载 / 卸载路径含**双斜杠** `//`                            | `detailDisk`、`attachDisk`、`detachDisk`           | Feign 原样传输，不影响功能，禁止修改路径                            |
| 5 | 规格查询为 POST，Body 内容全部参与加签，**null 值也不可省略**                | `queryFlavors`                                   | 加签时使用 `beanToMapWithInheritRelation(..., true)` 含 null 序列化 |
| 6 | 删除子网路径前缀为 `/v2`，其余子网接口前缀为 `/resource/v2`，两者不一致           | `deleteSubnet`                                   | 按联通云7原始路径实现，调用时无需特殊处理                              |
| 7 | `createNsg.projectId` 注释写"必须"，实际 2024-08-13 确认为非必填        | `createNsg`                                      | 按非必填处理，可传 null                                     |
| 8 | VM VNC 响应字段名为 `remote_console`（下划线命名），Java 默认驼峰无法自动反序列化 | `vmVnc`                                          | 已通过 `@JSONField(name = "remote_console")` 处理       |
| 9 | 裸金属创建 `network.vpcId` 实为子网的 `networkId`，非 VPC `instanceId` | `createBm`                                       | 字段语义特殊：需传子网对象中的 `networkId` 字段值                     |

---

*本文档由 Claude Code 基于代码分析自动生成，接口细节以联通云7官方 OpenAPI 文档为准。*
