# 🧠 JobWhiz - 智能职位追踪与模拟面试平台（完整项目文档）

---

## 🖼️ 一、前端设计（基于 Lovable）

### 1. 首页 `/`
#### 功能模块：
- 🔍 搜索栏（关键词 + 时间范围：1h / 24h）
- ⏱️ 时间过滤选择器（触发 `/jobs` API）
- 📋 岗位列表卡片展示：
  - 公司名 / 岗位 / 发布时间
  - [查看详情] ➝ 跳转 `/job/{jobId}`
  - [收藏 ❤️] ➝ POST `/user/favorite`
  - [已投递] ➝ 弹窗确认 ➝ POST `/application`
  - [🎤 Practice with AMA] ➝ 跳转至 AMA 模拟面试页面

---

### 2. 岗位详情页 `/job/{jobId}`
#### 模块：
- 职位标题 / 公司 / 地点
- 原始描述（来自 `/job/{jobId}`）
- AI 摘要（首次访问触发 `/job/{jobId}/summary`）

---

### 3. 追踪页 `/track`
#### 模块：
- 表格展示用户所有申请记录（来自 `/application/{userId}`）
- 可编辑状态（PUT `/application/{id}/status`）
- 可添加备注（PUT `/application/{id}/note`）
- 投递数据可视化图表（来自 `/application/stats`）

---

## 🧩 二、后端架构（Spring Boot + Redis + Kafka + DynamoDB + Lambda）

### 📐 架构概览：

```
Python 爬虫 ➝ Kafka topic: job-ingest ➝
Spring Boot Consumer ➝ 存入 Redis & DynamoDB
                                  ▲
    用户搜索请求 ➝ 查 Redis 缓存，不存在触发 Kafka ➝ 等待结果
```

---

## 📘 三、API 一览（含技术点）

| API | 方法 | 功能 | 技术 | 输入 / 输出 |
|------|------|------|------|-------------|
| `/jobs` | GET | 搜索岗位 | Redis + Kafka | kw, range | 岗位列表 |
| `/job/{jobId}` | GET | 岗位详情 | DynamoDB | jobId | 岗位信息 |
| `/job/{jobId}/summary` | GET | AI 摘要 | Lambda + GPT | jobId | 段落摘要 |
| `/job/{jobId}/ama-link` | GET | AMA 跳转链接 | URL 构造 | jobId | AMA URL |
| `/application` | POST | 创建投递记录 | DynamoDB | userId, jobId | OK |
| `/application/{userId}` | GET | 查询投递记录 | DynamoDB | userId | list |
| `/application/{id}/status` | PUT | 更新状态 | DynamoDB | status | OK |
| `/application/{id}/note` | PUT | 添加备注 | DynamoDB | text | OK |
| `/application/stats` | GET | 统计数据 | 聚合 | userId | JSON |
| `/user/favorite` | POST | 收藏岗位 | Redis + DB | jobId | OK |
| `/user/favorite/list` | GET | 收藏列表 | DB | userId | list |
| `/user/subscribe` | POST | 添加订阅关键词 | DB | kw list | OK |
| `/internal/job/push` | POST | 爬虫推送岗位数据 | Kafka + DB | job JSON | OK |
| `/internal/job/refresh-cache` | POST | 强制刷新缓存 | Redis + Kafka | kw | OK |

---

## 🗃️ 四、数据库设计（DynamoDB）

### 1. Job 表（岗位信息）
| 字段 | 类型 | 描述 |
|------|------|------|
| `jobId` | string | 主键 |
| `title` | string | 职位名称 |
| `company` | string | 公司名 |
| `location` | string | 地点 |
| `description` | string | 岗位内容 |
| `source` | string | 来源网站 |
| `postedAt` | datetime | 发布时间 |

---

### 2. Application 表（投递记录）
| 字段 | 类型 | 描述 |
|------|------|------|
| `applicationId` | string | 主键 |
| `userId` | string | 用户匿名 ID |
| `jobId` | string | 对应岗位 ID |
| `appliedAt` | datetime | 投递时间 |
| `status` | string | applied / interview / offer |
| `note` | string | 用户备注 |
| `source` | string | 来源网站 |
| `autoTracked` | boolean | 是否自动追踪状态 |

---

### 3. Favorite 表（收藏记录）
| 字段 | 类型 |
|------|------|
| `favoriteId` | string |
| `userId` | string |
| `jobId` | string |
| `favoritedAt` | datetime |

---

### 4. Subscription 表（关键词订阅）
| 字段 | 类型 |
|------|------|
| `subscriptionId` | string |
| `userId` | string |
| `keywords` | list<string> |
| `createdAt` | datetime |
