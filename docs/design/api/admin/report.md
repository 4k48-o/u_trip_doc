# 报表 — `/api/v1/admin/reports`

### 15.1 经营大屏

**GET** `/api/v1/admin/reports/dashboard`

**响应示例：**
```json
{
  "data": {
    "today": {
      "visitorCount": 8520,
      "yesterdayCompare": "+12%",
      "onlineRatio": "68%",
      "ticketRevenue": "340800.00",
      "totalRevenue": "562400.00"
    },
    "realTime": {
      "currentInPark": 3240,
      "ticketSold": { "08-10": 1200, "10-12": 2400, "12-14": 1800 },
      "gateStats": [
        { "gateName": "南口", "verified": 4520, "waiting": 120 },
        { "gateName": "北口", "verified": 4000, "waiting": 80 }
      ]
    },
    "ranking": {
      "otaTop5": ["携程 3200", "美团 2100", "抖音 1500", "去哪儿 800", "小红书 500"],
      "agentTop5": ["中国国旅 1200", "中青旅 800", "..."]
    },
    "visitorProfile": {
      "domesticRatio": "82%",
      "sourceTop5": ["北京 35%", "河北 15%", "天津 10%", "上海 8%", "广东 6%"],
      "genderRatio": { "male": "48%", "female": "52%" },
      "ageDistribution": { "0-18": "12%", "19-35": "38%", "36-55": "32%", "56+": "18%" }
    }
  }
}
```

### 15.2 售票报表

**GET** `/api/v1/admin/reports/ticket?startDate=X&endDate=Y&granularity=day/hour`

**响应：** 按日期/时段的售出数量 + 金额 + 渠道占比趋势。

### 15.3 营收报表

**GET** `/api/v1/admin/reports/revenue?startDate=X&endDate=Y&dimension=product/channel/seller`

### 15.4 游客画像分析

**GET** `/api/v1/admin/reports/visitor-profile?startDate=X&endDate=Y`

**响应：** 客源地分布、年龄性别、客单价区间、平均停留时长、复游率。

### 15.5 销售核销差异报表

**GET** `/api/v1/admin/reports/diff?date=X`

> 按核销日期做账时，销售数 vs 核销数的差异分析。

### 15.6 退票专题分析

**GET** `/api/v1/admin/reports/refund?startDate=X&endDate=Y`

**响应：** 退票数量、退票金额、退票率、退票原因分布、强制退票占比。

### 15.7 自定义报表

**POST** `/api/v1/admin/reports/custom`

> 通过 JimuReport 自定义报表查询。

| 字段 | 必填 | 说明 |
|------|------|------|
| reportId | 是 | 报表模板 ID |
| params | 否 | 参数 Map |

### 15.8 报表导出

**GET** `/api/v1/admin/reports/{type}/export`

> 各类型报表导出为文件。

| 参数 | 必填 | 说明 |
|------|------|------|
| type | 是 | ticket / revenue / visitor-profile / diff / refund |
| format | 是 | xlsx / pdf |
| startDate | 是 | 开始日期 |
| endDate | 是 | 结束日期 |
| ...其他报表查询参数 | 否 | 同各报表查询参数 |

**响应：** `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` 或 `application/pdf`

---

