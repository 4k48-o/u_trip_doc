# 旅行社端 API — `/api/v1/agent/`

> 面向旅行社操作员。需要旅行社 JWT Token。按旅行社分组类型限制可购产品和价格。

---

## 1. 产品与下单

### 1.1 可预订产品

**GET** `/api/v1/agent/products`

> 返回当前旅行社可预订的产品列表，价格按旅行社分组类型展示。

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "records": [{
      "spuId": "SPU001",
      "spuName": "绥中长城成人票",
      "categoryCode": "TICKET",
      "myPrice": "35.00",
      "originalPrice": "40.00",
      "myDiscount": "8.75折",
      "comboRestriction": "禁止单独购买门票，需组合车辆产品",
      "skus": [{
        "skuId": "SKU001",
        "skuName": "成人全价票",
        "sellPrice": "35.00",
        "maxBuyPerOrder": 200,
        "availableStock": 500
      }]
    }],
    "pageNo": 1
  }
}
```

---

### 1.2 产品详情与价格

**GET** `/api/v1/agent/products/{spuId}` — 完整商品信息 + 旅行社专属价格

---

### 1.3 下单（单团）

**POST** `/api/v1/agent/orders`

> 支持一人一码和一码多人两种模式。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| items | array | 是 | 订单项（同游客端格式） |
| contactName | string | 是 | 联系人 |
| contactPhone | string | 是 | 联系电话 |
| visitDate | date | 是 | 出行日期 |
| qrMode | string | 是 | per_person（一人一码）/ per_group（一码多人） |
| payMethod | string | 是 | qrcode / transfer / credit / on_site |
| remark | string | 否 | 备注 |

**items[].visitors** 最多 200 人。

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "orderId": "ORD20260601001",
    "orderNo": "MTY06010001",
    "totalAmount": "7000.00",
    "payMethod": "credit",
    "creditRemaining": "43000.00",
    "status": 2,
    "statusText": "已支付（授信扣款）"
  }
}
```

---

### 1.4 批量导入下单

**POST** `/api/v1/agent/orders/batch`

> 上传 Excel 文件批量导入团队游客信息。支持 200+ 人。

**Content-Type:** multipart/form-data

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| file | file | 是 | Excel 文件（.xlsx） |
| skuId | string | 是 | 票种 SKU ID |
| visitDate | string | 是 | 出行日期 |
| timeSlot | string | 是 | 时段 |
| payMethod | string | 是 | — |
| qrMode | string | 是 | per_person / per_group |

**Excel 模板列：** 序号 / 姓名 / 证件类型 / 证件号 / 国籍 / 性别 / 年龄

**响应示例（同步校验结果）：**
```json
{
  "code": 0,
  "data": {
    "batchNo": "BATCH06010001",
    "totalVisitors": 200,
    "successCount": 195,
    "failCount": 5,
    "failDetail": [
      { "row": 3, "name": "王五", "reason": "证件号重复" },
      { "row": 12, "name": "赵六", "reason": "年龄不满足购买条件（要求19-59岁，实际17岁）" }
    ],
    "orderId": "ORD20260601002",
    "orderNo": "MTY06010002"
  }
}
```

---

### 1.5 订单管理

**GET** `/api/v1/agent/orders`

| 参数 | 必填 | 说明 |
|------|------|------|
| orderNo | 否 | 订单号 |
| status | 否 | 1待支付/2已支付/3已核销/4已退款/5已取消 |
| travelDate | 否 | 出行日期 |
| pageNo / pageSize | — |

**响应records 额外字段：** visitorCount, verifyCount, payMethodText。

---

**GET** `/api/v1/agent/orders/{orderId}` — 订单详情（含游客列表 + 核销状态 + 凭证）

**POST** `/api/v1/agent/orders/{orderId}/cancel` — 取消订单（仅限未支付/未核销）

---

### 1.6 批量导入结果查询

**GET** `/api/v1/agent/batch/{batchNo}/result` — 查看批量导入处理结果

**GET** `/api/v1/agent/batch/{batchNo}/errors` — 下载失败明细 Excel

---

## 2. 账户管理

### 2.1 账户概览

**GET** `/api/v1/agent/account`

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "balance": "25000.00",
    "creditLimit": "50000.00",
    "usedCredit": "7000.00",
    "availableCredit": "43000.00",
    "totalRecharge": "30000.00",
    "totalConsumption": "12000.00",
    "payMethod": "credit",
    "status": 1
  }
}
```

---

### 2.2 在线充值

**POST** `/api/v1/agent/account/recharge`

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| amount | string | 是 | 充值金额（元） |
| payMethod | string | 是 | qrcode / transfer |

**响应：** 返回支付参数（同游客端支付）。充值成功后 `balance += amount`。

---

### 2.3 交易流水

**GET** `/api/v1/agent/account/transactions`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| transactionType | string | 否 | recharge / deduct / refund / adjust |
| startDate | string | 否 | 开始时间 |
| endDate | string | 否 | 结束时间 |
| pageNo | int | 否 | — |

**响应records字段：** transactionNo, transactionType, transactionTypeText, amount, balanceAfter, refId, remark, createTime。

---

### 2.4 对账单

**GET** `/api/v1/agent/account/ledgers`

> 按账期导出对账单（Excel）。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| periodStart | string | 是 | 账期开始 |
| periodEnd | string | 是 | 账期结束 |

**响应：**
```json
{
  "code": 0,
  "data": {
    "downloadUrl": "https://cdn.example.com/ledgers/agent_001_202605.xlsx?sign=...",
    "summary": { "totalOrders": 85, "totalAmount": "42000.00", "totalVerify": 83 }
  }
}
```

---

## 3. 通知与消息

### 3.1 订单通知

**GET** `/api/v1/agent/notifications`

> 协议到期提醒、订单状态变更、核销通知、系统公告等。

| 参数 | 必填 | 说明 |
|------|------|------|
| readFlag | 否 | 0未读/1已读 |

---

**POST** `/api/v1/agent/account/credit-apply` — 申请调整授信额度

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| applyAmount | string | 是 | 申请额度 |
| reason | string | 是 | 申请理由 |

**响应：** `{ "data": { "applyId": "...", "status": 0, "statusText": "待审核" } }`

**GET** `/api/v1/agent/contract` — 当前合同/协议信息

**响应：** agencyName, groupType, contractStartDate, contractEndDate, remainingDays, status, expireWarning(是否到期预警)。
