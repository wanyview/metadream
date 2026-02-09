# 交付物3：模块与API清单

## 文档概述

本文档详细定义造梦府DreamLearn系统的所有模块边界、API接口、硬件对接规范。可直接用于开发团队接口定义、硬件供应商技术对接。

---

## 1. 系统模块总览

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            模块关系总图                                           │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                         用户层（User Layer）                             │   │
│  │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │   │
│  │   │   移动App   │  │   Web控制台  │  │   硬件App   │  │   唤醒器   │  │   │
│  │   │  (iOS/And)  │  │   (Admin)   │  │  (配网/调试) │  │  (紧急)    │  │   │
│  │   └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  │   │
│  └──────────┼────────────────┼────────────────┼────────────────┼──────────┘   │
│             │                │                │                │              │
│             └────────────────┴────────────────┴────────────────┘              │
│                                    │                                          │
│                                    ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                       网关层（Gateway Layer）                            │   │
│  │   ┌─────────────────────────────────────────────────────────────────┐   │   │
│  │   │                    API Gateway                                  │   │   │
│  │   │   认证 / 限流 / 路由 / 日志 / 监控                               │   │   │
│  │   └─────────────────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                    │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                       业务服务层（Service Layer）                        │   │
│  │                                                                          │   │
│  │   ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │   │
│  │   │ 用户服务 │ │ 内容服务 │ │ 学习服务 │ │ 设备服务 │ │ 数据服务 │    │   │
│  │   │  /auth  │ │ /content │ │ /learn   │ │ /device │ │ /data   │    │   │
│  │   └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘    │   │
│  │        │            │            │            │            │           │   │
│  │        └────────────┴────────────┴────────────┴────────────┘           │   │
│  │                                    │                                     │   │
│  └────────────────────────────────────┼─────────────────────────────────────┘   │
│                                       │                                        │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                       AI服务层（AI Service Layer）                        │   │
│  │                                                                          │   │
│  │   ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │   │
│  │   │ 睡眠AI  │ │ 梦境AI  │ │ 推荐AI  │ │ 分析AI  │ │ 审核AI  │    │   │
│  │   │ /sleep  │ │ /dream  │ │ /recomm  │ │ /analytics│ │ /review │    │   │
│  │   └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘    │   │
│  │        │            │            │            │            │           │   │
│  │        └────────────┴────────────┴────────────┴────────────┘           │   │
│  │                                    │                                     │   │
│  └────────────────────────────────────┼─────────────────────────────────────┘   │
│                                       │                                        │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                       基础设施层（Infrastructure Layer）                  │   │
│  │                                                                          │   │
│  │   ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │   │
│  │   │ 消息队列 │ │ 缓存     │ │ 时序DB   │ │ 对象存储 │ │ 计算资源 │    │   │
│  │   │ (Kafka) │ │ (Redis)  │ │(InfluxDB)│ │  (S3)   │ │(K8s/GPU) │    │   │
│  │   └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘    │   │
│  │                                                                          │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. 模块详细定义

### 2.1 M01：睡眠监测与REM识别模块

| 属性 | 值 |
|-----|-----|
| **模块编号** | M01 |
| **模块名称** | SleepStaging |
| **核心功能** | 实时睡眠分期、REM窗口预测 |
| **部署位置** | 边缘设备 |
| **输入** | 原始EEG/EOG/ECG信号 |
| **输出** | 睡眠阶段标签、置信度 |

#### 2.1.1 内部子模块

```
M01-SUB1: SignalPreprocessor     // 信号预处理
M01-SUB2: FeatureExtractor       // 特征提取
M01-SUB3: SleepClassifier         // 睡眠分类器
M01-SUB4: REMWindowPredictor     // REM窗口预测
```

#### 2.1.2 内部接口

| 接口ID | 输入 | 处理 | 输出 |
|-------|------|------|------|
| M01-IF01 | 原始信号Buffer | 滤波/去噪 | 清洁信号 |
| M01-IF02 | 清洁信号 | FFT/小波变换 | 频谱特征 |
| M01-IF03 | 频谱特征+时序特征 | CNN-LSTM推理 | 睡眠阶段 |
| M01-IF04 | 历史睡眠数据 | Transformer推理 | REM窗口预测 |

---

### 2.2 M02：清醒梦诱导引擎

| 属性 | 值 |
|-----|-----|
| **模块编号** | M02 |
| **模块名称** | LucidDreamInduction |
| **核心功能** | 多模态诱导方案生成与执行 |
| **部署位置** | 边缘设备 |
| **输入** | REM窗口信号、内容指令 |
| **输出** | 多模态刺激信号 |

#### 2.2.1 诱导方案类型

| 方案名称 | 适用场景 | 核心参数 |
|---------|---------|---------|
| MILD-Basic | 一般诱导 | 声学+视觉弱诱导 |
| MILD-Plus | 强化诱导 | 声学+视觉+触觉 |
| SWILD | 快速眼动诱导 | 视觉闪烁+轻触觉 |
| Audio-Remap | 内容强化 | 关键词音频+叙事 |

---

### 2.3 M03：内容神经编码编译器

| 属性 | 值 |
|-----|-----|
| **模块编号** | M03 |
| **模块名称** | NeuralEncoder |
| **核心功能** | 学习内容→梦境信号转换 |
| **部署位置** | 云端+边缘 |
| **输入** | 结构化内容、用户模型 |
| **输出** | 神经编码信号 |

#### 2.3.1 支持内容类型

| 内容类型 | 编码格式 | 示例 |
|---------|---------|------|
| 历史叙事 | 场景序列+锚点 | 安史之乱场景 |
| 语言对话 | 音频+关键词 | 雅思口语场景 |
| 考点速记 | 视觉符号+口诀 | 数学公式 |
| 技能流程 | 步骤序列+触觉 | 钢琴指法 |

---

### 2.4 M04：个性化适配AI

| 属性 | 值 |
|-----|-----|
| **模块编号** | M04 |
| **模块名称** | PersonalizationEngine |
| **核心功能** | 用户模型迭代、内容推荐 |
| **部署位置** | 云端 |
| **输入** | 学习记录、反馈数据 |
| **输出** | 更新后的用户模型 |

---

### 2.5 M05：醒后复盘系统

| 属性 | 值 |
|-----|-----|
| **模块编号** | M05 |
| **模块名称** | PostSleepReview |
| **核心功能** | 报告生成、卡片创建、复习提醒 |
| **部署位置** | 云端 |
| **输入** | 睡眠数据、学习事件 |
| **输出** | 学习报告、巩固卡片 |

---

### 2.6 M06：未成年人安全模式

| 属性 | 值 |
|-----|-----|
| **模块编号** | M06 |
| **模块名称** | MinorSafetyMode |
| **核心功能** | 内容过滤、时长管控 |
| **部署位置** | 云端+边缘 |
| **输入** | 用户年龄、学习请求 |
| **输出** | 过滤后的内容/限制响应 |

---

### 2.7 M07：数据安全模块

| 属性 | 值 |
|-----|-----|
| **模块编号** | M07 |
| **模块名称** | DataSecurity |
| **核心功能** | 加密、隐私保护、访问控制 |
| **部署位置** | 全栈 |
| **输入** | 用户数据、敏感信息 |
| **输出** | 加密数据、访问令牌 |

---

### 2.8 M08：实验数据面板

| 属性 | 值 |
|-----|-----|
| **模块编号** | M08 |
| **模块名称** | ExperimentDashboard |
| **核心功能** | 研究者数据可视化 |
| **部署位置** | 云端Web |
| **输入** | 聚合统计数据 |
| **输出** | 可视化图表、报表 |

---

## 3. REST API清单

### 3.1 用户服务 (/api/v1/auth)

| 方法 | 路径 | 功能 | 请求参数 | 响应 |
|-----|------|------|---------|------|
| POST | /api/v1/auth/register | 用户注册 | email, password, name | token, userId |
| POST | /api/v1/auth/login | 登录 | email, password | token, refreshToken |
| POST | /api/v1/auth/logout | 登出 | (空) | {success: true} |
| POST | /api/v1/auth/refresh | 刷新Token | refreshToken | newToken |
| GET | /api/v1/auth/profile | 获取信息 | (空) | userProfile |
| PUT | /api/v1/auth/profile | 更新信息 | profileData | updatedProfile |
| POST | /api/v1/auth/child/add | 添加未成年 | childInfo | childId |
| GET | /api/v1/auth/children | 子女列表 | (空) | [children] |

### 3.2 内容服务 (/api/v1/content)

| 方法 | 路径 | 功能 | 请求参数 | 响应 |
|-----|------|------|---------|------|
| GET | /api/v1/content/list | 内容列表 | category, page, size | contentList |
| GET | /api/v1/content/{id} | 内容详情 | contentId | contentDetail |
| POST | /api/v1/content/preview | 预览内容 | contentId | previewData |
| GET | /api/v1/content/categories | 分类列表 | (空) | categories |
| GET | /api/v1/content/search | 搜索内容 | keyword, filters | searchResults |

### 3.3 学习服务 (/api/v1/learn)

| 方法 | 路径 | 功能 | 请求参数 | 响应 |
|-----|------|------|---------|------|
| POST | /api/v1/learn/sessions | 创建学习会话 | contentId, intensity | sessionId |
| GET | /api/v1/learn/sessions/{id} | 会话详情 | sessionId | sessionDetail |
| GET | /api/v1/learn/sessions/{id}/report | 学习报告 | sessionId | learningReport |
| POST | /api/v1/learn/feedback | 提交反馈 | sessionId, feedback | {success: true} |
| GET | /api/v1/learn/history | 学习历史 | page, size | historyList |
| GET | /api/v1/learn/consolidation/{id} | 巩固卡片 | cardId | consolidationCard |

### 3.4 设备服务 (/api/v1/device)

| 方法 | 路径 | 功能 | 请求参数 | 响应 |
|-----|------|------|---------|------|
| POST | /api/v1/device/pair | 设备配对 | deviceInfo | pairingCode |
| POST | /api/v1/device/{id}/bind | 绑定设备 | deviceId, userId | {success: true} |
| GET | /api/v1/device/{id}/status | 设备状态 | deviceId | statusData |
| POST | /api/v1/device/{id}/command | 发送指令 | deviceId, command | {success: true} |
| GET | /api/v1/device/list | 设备列表 | (空) | [devices] |
| DELETE | /api/v1/device/{id} | 解绑设备 | deviceId | {success: true} |

### 3.5 数据服务 (/api/v1/data)

| 方法 | 路径 | 功能 | 请求参数 | 响应 |
|-----|------|------|---------|------|
| GET | /api/v1/data/sleep/{date} | 睡眠数据 | date | sleepData |
| GET | /api/v1/data/sleep/summary | 睡眠摘要 | startDate, endDate | summary |
| POST | /api/v1/data/export | 导出数据 | format, dateRange | downloadUrl |
| POST | /api/v1/data/delete | 删除数据 | dataTypes | {success: true} |

---

## 4. WebSocket API

### 4.1 实时信号通道

| 通道 | 方向 | 功能 | 消息格式 |
|-----|------|------|---------|
| /ws/signal | 设备→服务器 | 上传原始信号 | Binary Frame (EEG数据) |
| /ws/stage | 服务器→设备 | 推送睡眠阶段 | JSON (分期结果) |
| /ws/command | 服务器→设备 | 推送诱导指令 | JSON (诱导参数) |
| /ws/feedback | 设备→服务器 | 上报执行状态 | JSON (确认消息) |

### 4.2 消息格式定义

```json
// /ws/stage 消息格式
{
  "type": "sleep_stage",
  "timestamp": "2026-02-09T13:42:00.000Z",
  "stage": "REM",
  "confidence": 0.92,
  "rem_probability": 0.88
}

// /ws/command 消息格式
{
  "type": "induction_command",
  "command_id": "cmd_001",
  "timestamp": "2026-02-09T13:42:00.000Z",
  "induction_type": "mild_plus",
  "parameters": {
    "audio": {...},
    "visual": {...},
    "haptic": {...}
  },
  "content_id": "hist_anlush_01",
  "priority": "high"
}

// /ws/feedback 消息格式
{
  "type": "command_feedback",
  "command_id": "cmd_001",
  "status": "executed",
  "execution_time_ms": 150,
  "error": null
}
```

---

## 5. 硬件对接接口

### 5.1 EEG采集设备接口

| 参数 | 值 | 说明 |
|-----|-----|------|
| **接口协议** | Bluetooth LE 5.0 / USB HID | 双模式 |
| **数据格式** | Binary (Int16) | 16位有符号整数 |
| **采样率** | 256Hz (可配置) | 可选128/256/512Hz |
| **通道数** | 8-16通道 | 可配置 |
| **数据帧** | 64字节/帧 | 含时间戳+CRC |
| **功耗** | <500mW | 电池供电 |
| **延迟** | <50ms | 传输延迟 |

#### 5.1.1 数据帧格式

```
┌─────────────────────────────────────────────────────────────────┐
│                     EEG数据帧 (64 bytes)                        │
├─────────────────────────────────────────────────────────────────┤
│  Offset │  Size │  Field          │  Description                │
├─────────┼───────┼─────────────────┼────────────────────────────┤
│    0    │   4   │  timestamp      │  Unix timestamp (ms)        │
│    4    │   2   │  seq_num        │  序列号                      │
│    6    │   1   │  num_channels   │  通道数 (n)                 │
│    7    │   1   │  flags          │  状态标志位                  │
│    8    │  2n  │  samples[n]     │  各通道采样值 (Int16)       │
│  8+2n   │   4   │  crc32          │  校验和                      │
│ 12+2n   │  52   │  reserved       │  保留字段                    │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 诱导设备接口

| 设备类型 | 接口协议 | 控制方式 | 参数范围 |
|---------|---------|---------|---------|
| LED视觉诱导 | Bluetooth LE | PWM调制 | 亮度0-100%, 频率0.1-10Hz |
| 音频诱导 | Bluetooth A2DP | 音频流 | 音量0-100%, 频率可配置 |
| 振动马达 | Bluetooth LE | PWM控制 | 强度0-100%, 频率0.1-5Hz |
| 香薰机 | Bluetooth LE | 开关控制 | 开/关, 浓度可调 |

#### 5.2.1 诱导指令协议

```json
// 视觉诱导指令
{
  "device_type": "visual_inducer",
  "command": "start",
  "parameters": {
    "wavelength_nm": 530,      // 波长 (nm)
    "brightness_percent": 30, // 亮度 (%)
    "pulse_freq_hz": 0.5,      // 脉冲频率 (Hz)
    "pattern": "breathing",   // 模式: constant/pulse/breathing
    "duration_sec": 300       // 持续时间 (s)
  }
}

// 音频诱导指令
{
  "device_type": "audio_inducer",
  "command": "play",
  "parameters": {
    "audio_type": "theta_beat",  // 类型: white_noise/pink_noise/theta_beat
    "base_freq_hz": 6.0,         // 基频
    "volume_percent": 40,        // 音量
    "modulation": "AM",          // 调制方式
    "duration_sec": 300
  }
}

// 振动诱导指令
{
  "device_type": "haptic_inducer",
  "command": "vibrate",
  "parameters": {
    "pattern": "pulse",
    "frequency_hz": 0.5,
    "amplitude_percent": 30,
    "duration_sec": 300
  }
}
```

### 5.3 边缘计算设备规格

| 配置项 | 最低配置 | 推荐配置 |
|-------|---------|---------|
| 处理器 | Raspberry Pi 5 | Jetson Nano 4GB |
| CPU | 4-core Cortex-A76 | 4-core ARM57 |
| RAM | 4GB | 8GB |
| 存储 | 32GB eMMC | 64GB NVMe SSD |
| NPU | 可选 | 内置 |
| 功耗 | 5-10W | 10-15W |
| 操作系统 | Ubuntu Server | Ubuntu Desktop |
| 连接 | WiFi 6 / BT 5.0 | WiFi 6 / BT 5.0 |

---

## 6. 云端服务接口

### 6.1 AI服务接口

| 服务 | 接口 | 方法 | 输入 | 输出 |
|-----|------|------|------|------|
| 睡眠分期 | /api/v1/ai/sleep/stage | POST | eeg_data | {stage, confidence} |
| REM预测 | /api/v1/ai/sleep/rem-predict | POST | sleep_history | {windows[]} |
| 梦境生成 | /api/v1/ai/dream/generate | POST | content_request | {narrative_segments[]} |
| 内容推荐 | /api/v1/ai/recommend | POST | user_context | {recommendations[]} |
| 效果分析 | /api/v1/ai/analytics/session | POST | session_data | {metrics{}} |

### 6.2 消息队列接口

| 队列名 | 用途 | 消息格式 | 消费频率 |
|-------|------|---------|---------|
| sleep_staging | 睡眠分期结果 | JSON | 实时 |
| induction_events | 诱导事件 | JSON | 实时 |
| learning_events | 学习事件 | JSON | 实时 |
| analytics_data | 分析数据 | JSON | 批量 |
| alerts | 告警信息 | JSON | 实时 |

---

## 7. SDK与开发者工具

### 7.1 设备SDK

| SDK | 语言 | 用途 |
|-----|------|------|
| dreamlearn-ble-sdk | Python/C++ | BLE设备通信 |
| dreamlearn-usb-sdk | C/C++ | USB设备通信 |
| dreamlearn-ws-client | JavaScript | WebSocket连接 |

### 7.2 内容开发工具

| 工具 | 用途 |
|-----|------|
| DreamEditor | 可视化梦境内容编辑器 |
| ContentValidator | 内容合规性检查工具 |
| SimulationKit | 梦境场景模拟测试工具 |

---

## 8. 错误码定义

| 错误码 | 说明 | 处理建议 |
|-------|------|---------|
| 10001 | 信号质量差 | 检查传感器连接 |
| 10002 | 设备离线 | 检查设备电源/连接 |
| 10003 | 睡眠分期失败 | 重试或人工确认 |
| 20001 | 内容不存在 | 检查内容ID |
| 20002 | 内容未授权 | 检查订阅状态 |
| 30001 | 诱导超时 | 重试诱导 |
| 30002 | 诱导冲突 | 等待当前诱导结束 |
| 40001 | 用户未登录 | 跳转登录 |
| 40002 | 未成年人限制 | 检查年龄设置 |

---

*文档版本：v1.0*  
*作者：造梦府系统架构组*  
*最后更新：2026-02-09*
