<p align="center">
  <img width="144" alt="屏幕截图 2024-11-01 152350" src="https://github.com/user-attachments/assets/a9eb6c84-d094-4a3c-8bde-e33085da169f">
</p>

<h1 align="center">PistisChain</h1>

## 项目概述 
本项目实现一个区块链学生签到系统，通过学生节点的竞争挖矿来记录出勤数据，使用动态难度调整和分叉解决机制确保链的一致性和公平性。系统包括教师和学生两种角色，不同角色具有不同权限。前端实现学生和教师的操作界面，后端实现区块链和出勤记录管理，节点之间通过 HTTP 模拟 P2P 通信来同步数据。
<p><a href="https://github.com/Vine-Armor-Legion/frontend">前端项目地址</a></p>

## 技术栈 

### 前端 
 
- **框架** ：React
 
- **组件库** ：Ant Design
 
- **状态管理** ：React Context（或 Redux）
 
- **通信协议** ：HTTP 请求

### 后端 
 
- **框架** ：Express（暂定）
 
- **数据库** ：MySQL 或 MongoDB
 
- **通信协议** ：HTTP（使用 `superagent` 库）

### 节点通信 
 
- **方式** ：基于 HTTP 模拟 P2P 通信
 
- **库** ：`superagent` 进行节点间的 HTTP 请求

## 功能设计 

### 1. 学生和教师角色划分 

#### 学生角色 
 
- **签到和出勤记录** ：学生通过前端界面提交出勤记录，数据通过签名后存储在区块链中。
 
- **挖矿** ：学生可以通过前端界面触发挖矿操作，参与区块打包，并获取挖矿奖励。
 
- **出勤和挖矿奖励** ：学生查看自己的签到和挖矿记录，以及挖矿奖励积分。

#### 教师角色 
 
- **查询学生出勤** ：教师可以通过前端界面查询所有学生的出勤记录。
 
- **数据管理权限** ：教师具有更高的查询权限，可以查询班级的整体出勤情况或单个学生的出勤记录。

### 2. 数据存储设计 

本项目将数据存储分为两部分，以确保链上和链下数据的高效管理：

#### 区块链数据（JSON 文件） 
 
- **区块链数据** ：区块链结构存储在 JSON 文件中，包括每个区块的索引、哈希、前一个区块的哈希、时间戳、挖矿难度、随机数（nonce）、交易数据等。
 
- **交易数据（出勤记录的哈希）** ：每个区块包含出勤记录的哈希值，用于验证链上记录的完整性。
  
- **优势** ：JSON 文件结构适合在节点间同步区块链数据，便于快速模拟区块链的去中心化结构。区块链数据的透明性和不可篡改性通过 JSON 文件存储和多节点维护得以保障。

#### 业务数据（数据库） 
 
- **用户信息** ：包括学生和教师的 ID、姓名、角色等基本信息，用于管理用户权限。
 
- **出勤记录详情** ：包含每个出勤记录的详细信息，如学生 ID、课程 ID、出勤时间等，便于查询和分析。
 
- **奖励积分** ：存储学生通过挖矿获得的奖励积分，用于学生的账户管理。

- **优势** ：数据库支持高效查询，适合存储和管理复杂的出勤记录、用户信息和挖矿奖励等，确保系统运行的高效性和便捷性。将详细业务数据存储在数据库中，可以在链上记录出勤的哈希，用于验证和确保数据的完整性，而链下数据可以灵活地支持系统的查询和管理需求。

### 数据库设计（MySQL/MongoDB） 

#### 用户表（users） 
| 字段名 | 数据类型 | 描述 | 
| --- | --- | --- | 
| id | INT | 用户 ID | 
| name | VARCHAR | 用户姓名 | 
| role | VARCHAR | 用户角色（教师或学生） | 
| publicKey | VARCHAR | 用户公钥 | 

#### 出勤记录表（attendance_records） 
| 字段名 | 数据类型 | 描述 | 
| --- | --- | --- | 
| id | INT | 出勤记录 ID | 
| studentId | INT | 学生 ID | 
| eventId | INT | 事件 ID | 
| timestamp | DATETIME | 出勤时间 | 
| transactionHash | VARCHAR | 出勤记录的哈希 | 

#### 挖矿奖励表（rewards） 
| 字段名 | 数据类型 | 描述 | 
| --- | --- | --- | 
| id | INT | 记录 ID | 
| studentId | INT | 学生 ID | 
| points | INT | 积分 | 

### 3. 区块链结构设计 

每个区块存储的基本数据包括：
 
- **索引** （index）：区块的序号
 
- **前一个区块的哈希** （previousHash）：指向链上上一个区块
 
- **当前区块的哈希** （hash）：该区块的数据哈希值
 
- **时间戳** （timestamp）：区块生成时间
 
- **随机数** （nonce）：用于工作量证明
 
- **难度** （difficulty）：动态调整的挖矿难度
 
- **交易数据** （transactions）：出勤记录的哈希

示例 JSON 数据结构：


```json
[
    {
        "index": 1,
        "timestamp": "2024-01-01T00:00:00Z",
        "previousHash": "0000000000",
        "hash": "abc123...",
        "nonce": 12345,
        "difficulty": 2,
        "transactions": [
            { "hash": "tx123hash", "studentId": "s1" }
        ]
    }
]
```

### 4. 区块链核心功能 

#### 挖矿和竞争 
 
1. **学生通过前端触发挖矿** ，后台执行 PoW 算法进行挖矿。
 
2. **动态调整挖矿难度** ：系统基于出块时间调整难度，保持出块时间稳定。
 
3. **区块广播** ：节点成功挖矿后，调用 `broadcast` 将新区块广播到所有节点。

#### 分叉处理 
 
- 采用**最长链** 或**最高累积难度链** 原则解决分叉问题。
 
- 接收到新区块后调用 `checkReceivedBlocks` 判断链的优劣，选择最长链或累积难度最高的链作为有效链。

#### 数据同步和广播 
 
- 使用 `superagent` 库的 HTTP 请求实现节点间的广播和数据同步，包括出勤记录和新区块的广播。
 
- 每个节点接收广播后，使用 `syncTransactions` 或 `checkReceivedBlocks` 确保链数据和出勤数据一致。

### 5. 前后端交互 

#### 前端操作（React + Ant Design） 
 
- **学生界面** ：签到按钮、挖矿按钮、出勤记录查看等功能。
 
- **教师界面** ：查看所有学生的出勤记录，筛选和统计出勤数据。

#### 后端 API 设计（Express） 
 
- **用户相关 API** ： 
  - `POST /api/login`：登录接口，获取用户权限。
 
  - `GET /api/user/:id`：获取用户信息。
 
- **出勤记录 API** ： 
  - `POST /api/attendance`：学生签到提交接口。
 
  - `GET /api/attendance/:studentId`：教师查看学生的出勤记录。
 
- **挖矿相关 API** ： 
  - `POST /api/mine`：触发挖矿操作。
 
  - `GET /api/chain`：获取区块链的当前状态。

### 6. 系统流程 
 
1. **学生登录并签到** ：
  - 学生登录系统，进入签到页面，点击签到按钮，将出勤记录上传至服务器。
 
2. **竞争挖矿** ：
  - 学生可点击挖矿按钮触发挖矿操作，系统启动 PoW 挖矿算法，成功挖矿后广播新区块。
 
3. **链同步和分叉解决** ：
  - 节点接收到新区块广播后判断链长度或累积难度，进行链数据更新。
 
4. **教师查询出勤记录** ：
  - 教师登录系统后，可以查看和筛选学生的出勤信息，并查询挖矿奖励等数据。

### 7. 数据存储方式总结 
 
- **JSON 文件** ：区块链数据，包括每个区块和出勤记录哈希。
 
- **数据库** ：出勤记录详情、用户信息和挖矿奖励。
