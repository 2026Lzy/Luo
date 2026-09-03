# 基于多智能算法的机械设备故障预测与运维决策系统

> 制造智能技术课程设计成果 | Vibe Coding（AI 辅助编程）开发模式

---

## 目录

- [项目简介](#项目简介)
- [项目背景](#项目背景)
- [核心功能](#核心功能)
- [系统架构](#系统架构)
- [技术栈](#技术栈)
- [数据来源](#数据来源)
- [项目结构](#项目结构)
- [快速开始](#快速开始)
- [API 接口说明](#api-接口说明)
- [模型性能](#模型性能)
- [前端性能优化](#前端性能优化)
- [开发过程](#开发过程)
- [贡献指南](#贡献指南)
- [许可证](#许可证)

---

## 项目简介

本项目为**制造智能技术课程设计**成果，面向工业设备运维场景，构建了完整的 **"状态感知 — 故障诊断 — 寿命预测 — 运维决策"** 智能业务闭环。系统采用 B/S 架构，集成机器学习故障分类、时序剩余使用寿命预测、启发式运维优化决策三大核心算法，覆盖课程至少 3 项技术方向，可直接运行演示。

项目采用 **Vibe Coding（AI 辅助编程）** 模式开发，全程留存开发过程档案，遵循小步 Git 提交规范，满足课程过程考核要求。

### 项目亮点

- **万级数据秒级处理**：支持 13096 条测试数据一次性导入，批量诊断 10 秒内出结果
- **双模型协同预测**：故障分类 + 剩余使用寿命（RUL）回归双模型并行推理
- **前后端分离架构**：Vue3 + FastAPI，RESTful API 设计，接口文档自动生成
- **轻量化部署**：CDN 引入前端依赖，SQLite 零配置数据库，普通笔记本即可运行

---

## 项目背景

工业设备意外故障停机是造成生产损失的核心原因，传统定期检修模式普遍存在过维修或欠维修问题。据统计，工业设备因突发故障导致的非计划停机损失占企业总维护成本的 30% 以上，而预测性维护可以降低维护成本 25%~30%，减少停机时间 70% 以上。

本项目以航空发动机退化场景为切入点，基于 NASA 公开工业数据集，运用制造智能课程所学技术，实现：

1. **智能故障诊断**：基于传感器数据自动识别设备故障状态（风扇故障、压气机故障、涡轮故障、组合故障）
2. **智能寿命预测**：预测设备剩余使用寿命（RUL）并分级预警
3. **智能运维决策**：多维度综合评估生成运维优先级与处置方案

---

## 核心功能

### 1. 数据导入模块

- 支持 CSV 格式设备运行数据文件上传
- Web Worker 后台线程解析，分块传输，万级数据不卡顿
- 实时显示样本总数、设备数量、特征维度等统计信息
- 样本数据脱离 Vue 响应式系统，通过版本号触发更新

### 2. 故障诊断模块

- **单样本诊断**：选择特定设备和运行周期，识别故障类型及置信度
- **批量诊断**：对全部导入数据批量分析，输出各样本诊断结果
- 置信度可视化：ECharts 柱状图展示各类别预测概率
- 远程搜索下拉框：万级样本通过关键词快速定位

### 3. 寿命预测模块

- 基于随机森林回归模型预测剩余使用寿命（RUL）
- 数值展示 + 传感器特征变化趋势图
- 标注设备当前退化阶段（早期稳定 / 中期退化 / 后期失效）

### 4. 运维决策模块

- 四级风险评估：正常（绿）/ 关注（黄）/ 警告（橙）/ 危险（红）
- 根据故障类型和 RUL 自动生成维护建议
- 建议维护时机、维护方式、预估停机时间和成本

### 5. 历史记录模块

- 自动保存每次分析结果到 SQLite 数据库
- 按设备编号、时间范围、故障类型筛选查询
- 支持记录删除和批量管理

---

## 系统架构

采用标准四层 B/S 架构，前后端分离，数据全链路持久化：

```
┌─────────────────────────────────────────────────┐
│              前端 UI 层（浏览器）                  │
│   Vue3 + Element Plus + ECharts（CDN 引入）      │
│   数据导入 │ 故障诊断 │ 寿命预测 │ 运维决策 │ 历史  │
└──────────────────────┬──────────────────────────┘
                       │ HTTP / RESTful API
┌──────────────────────▼──────────────────────────┐
│              后端服务层（FastAPI）                 │
│   Uvicorn ASGI 服务器 │ CORS 跨域 │ 全局异常处理  │
│   Pydantic 参数校验 │ Swagger UI 自动文档         │
└──────────────────────┬──────────────────────────┘
                       │ 模型调用
┌──────────────────────▼──────────────────────────┐
│              算法模型层（scikit-learn）            │
│   ┌──────────────┐    ┌──────────────────────┐  │
│   │ 故障分类模型   │    │ RUL 预测模型          │  │
│   │ RandomForest  │    │ RandomForest         │  │
│   │ Classifier    │    │ Regressor            │  │
│   │ 100 棵树      │    │ 200 棵树, max_depth=20│ │
│   └──────────────┘    └──────────────────────┘  │
│   StandardScaler 标准化 │ joblib 模型序列化        │
└──────────────────────┬──────────────────────────┘
                       │ 数据读写
┌──────────────────────▼──────────────────────────┐
│              数据持久层（SQLite）                  │
│   设备信息表 │ 传感器数据表 │ 故障诊断记录表       │
│   寿命预测记录表 │ 运维决策记录表                  │
└─────────────────────────────────────────────────┘
```

---

## 技术栈

### 前端

| 技术 | 版本 | 用途 |
|------|------|------|
| Vue3 | 3.3+ | 前端框架（CDN 引入，无需构建） |
| Element Plus | 2.3+ | UI 组件库 |
| ECharts | 5.4+ | 数据可视化图表 |
| Web Worker | - | 后台线程解析 CSV |
| Axios | - | HTTP 请求库 |

### 后端

| 技术 | 版本 | 用途 |
|------|------|------|
| Python | 3.10+ | 开发语言 |
| FastAPI | 0.100+ | Web 框架 |
| Uvicorn | 0.23+ | ASGI 服务器 |
| scikit-learn | 1.3+ | 机器学习算法 |
| Pandas | 2.0+ | 数据处理 |
| NumPy | - | 数值计算 |
| joblib | 1.3+ | 模型序列化 |
| Pydantic | - | 数据校验 |
| SQLite3 | - | 轻量级数据库 |

### 开发工具

| 工具 | 用途 |
|------|------|
| Git | 版本控制 |
| GitHub | 代码托管 |
| Vibe Coding | AI 辅助编程开发模式 |

---

## 数据来源

### 数据集基本信息

**数据集全称**：NASA C-MAPSS 涡扇发动机退化仿真数据集
（Commercial Modular Aero-Propulsion System Simulation，商用模块化航空推进系统仿真数据集）

该数据集由 NASA 艾姆斯研究中心发布，是工业预测性维护领域的标准公开数据集，模拟航空发动机从正常运行到性能退化直至故障的全生命周期时序数据，广泛用于故障诊断、剩余使用寿命（RUL）预测等制造智能技术研究。

### 官方来源与下载链接

1. **官方发布源（PHM 协会）**
   - 数据集官方主页：[https://data.phmsociety.org/nasa/](https://data.phmsociety.org/nasa/)
   - 原始数据集直接下载地址：[https://phm-datasets.s3.amazonaws.com/NASA/6.+Turbofan+Engine+Degradation+Simulation+Data+Set.zip](https://phm-datasets.s3.amazonaws.com/NASA/6.+Turbofan+Engine+Degradation+Simulation+Data+Set.zip)

2. **国内镜像源（推荐，下载速度更快）**
   - 魔搭 ModelScope 镜像：[https://modelscope.cn/datasets/AI4Manufacture/NASA_CMAPSS](https://modelscope.cn/datasets/AI4Manufacture/NASA_CMAPSS)
   - 飞桨 AI Studio 镜像：[https://aistudio.baidu.com/datasetdetail/11724](https://aistudio.baidu.com/datasetdetail/11724)

### 数据集统计

| 指标 | 训练集 | 测试集 | 合计 |
|------|--------|--------|------|
| 设备数量 | 100 | 100 | 200 |
| 样本数量 | 16,340 | 13,096 | 29,436 |
| 原始特征数 | 21 | 21 | 21 |
| 筛选后特征数 | 14 | 14 | 14 |

**筛选后的 14 个关键传感器特征**：
`s2, s3, s4, s7, s8, s9, s11, s12, s13, s14, s15, s17, s20, s21`

---

## 项目结构

```
Luo/
├── backend/                    # 后端服务
│   ├── app.py                 # FastAPI 主应用（路由、模型加载、CORS）
│   ├── api/                   # API 路由模块
│   │   ├── fault.py          # 故障诊断接口
│   │   ├── rul.py            # 寿命预测接口
│   │   └── maintenance.py    # 运维决策接口
│   └── models/               # 模型文件目录
│
├── algorithms/                 # 算法模块
│   ├── fault_classifier.py   # 故障分类算法
│   ├── rul_predictor.py      # RUL 预测算法
│   ├── model_train.py        # 模型训练脚本
│   └── models/               # 训练好的模型文件
│       ├── fault_classifier.pkl  # 故障分类模型（7.27 MB）
│       └── rul_predictor.pkl     # RUL 预测模型（88.09 MB）
│
├── frontend/                   # 前端页面
│   └── index.html            # 单页面应用（Vue3 + Element Plus + ECharts）
│
├── data/                       # 数据目录
│   ├── raw/                  # 原始数据
│   └── processed/            # 预处理后数据
│       ├── train.csv         # 训练集（16340 条）
│       └── test.csv          # 测试集（13096 条）
│
├── prompt/                     # JSON 配置与示例文件
│   ├── 2026-09-03_config.json
│   ├── 2026-09-03_sample_data.json
│   ├── 2026-09-03_fault_predict_request.json
│   ├── 2026-09-03_fault_predict_response.json
│   ├── 2026-09-03_rul_predict_request.json
│   ├── 2026-09-03_rul_predict_response.json
│   ├── 2026-09-03_batch_predict_request.json
│   ├── 2026-09-03_maintenance_decision.json
│   ├── 2026-09-03_api_endpoints.json
│   └── 2026-09-03_model_info.json
│
├── .gitignore                 # Git 忽略文件
└── README.md                  # 项目说明文档
```

---

## 快速开始

### 环境要求

- Python 3.10 或更高版本
- 现代浏览器（Chrome / Edge / Firefox）
- 至少 2GB 可用内存

### 1. 克隆仓库

```bash
git clone https://github.com/2026Lzy/Luo.git
cd Luo
```

### 2. 安装后端依赖

```bash
pip install fastapi uvicorn scikit-learn pandas numpy joblib pydantic
```

### 3. 训练模型（首次运行需要）

```bash
cd algorithms
python model_train.py
```

训练完成后，模型文件将生成在 `algorithms/models/` 目录下：
- `fault_classifier.pkl`（故障分类模型）
- `rul_predictor.pkl`（RUL 预测模型）

### 4. 启动后端服务

```bash
cd backend
python app.py
```

服务启动后，访问以下地址：
- 后端服务：http://localhost:8000
- 接口文档（Swagger UI）：http://localhost:8000/docs

### 5. 打开前端页面

直接用浏览器打开 `frontend/index.html` 文件即可，无需启动前端服务器。

> **注意**：如果浏览器提示跨域问题，请确保后端服务已启动，且 `app.py` 中已配置 CORS 允许所有来源。

### 一键启动（Windows）

```bat
@echo off
cd /d %~dp0
start cmd /k "cd backend && python app.py"
timeout /t 3
start "" "frontend\index.html"
```

---

## API 接口说明

所有接口基础地址：`http://localhost:8000`

### 统一响应格式

```json
{
  "code": 0,
  "message": "success",
  "data": { }
}
```

- `code`：0 表示成功，非 0 表示失败
- `message`：响应消息
- `data`：响应数据

### 接口列表

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/health` | 健康检查 |
| GET | `/docs` | Swagger UI 接口文档 |
| POST | `/api/fault/predict` | 单样本故障诊断 |
| POST | `/api/fault/batch` | 批量故障诊断 |
| POST | `/api/rul/predict` | 剩余使用寿命预测 |
| POST | `/api/maintenance` | 运维决策建议 |
| GET | `/api/history` | 历史记录查询 |
| GET | `/api/history/{id}` | 查询单条历史记录 |
| DELETE | `/api/history/{id}` | 删除历史记录 |
| GET | `/api/stats` | 系统统计信息 |

### 单样本故障诊断

**请求**：`POST /api/fault/predict`

```json
{
  "features": [643.52, 1590.87, 1403.14, 21.61, 553.97, 2388.03, 47.24, 521.48, 2388.05, 8126.34, 8.4238, 392, 38.98, 23.4283],
  "device_id": "engine_001",
  "cycle": 100
}
```

**响应**：

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "device_id": "engine_001",
    "cycle": 100,
    "fault_type": "压气机故障",
    "confidence": 0.87,
    "probabilities": {
      "正常": 0.05,
      "风扇故障": 0.03,
      "压气机故障": 0.87,
      "涡轮故障": 0.03,
      "组合故障": 0.02
    }
  }
}
```

### 剩余使用寿命预测

**请求**：`POST /api/rul/predict`

```json
{
  "features": [643.52, 1590.87, 1403.14, 21.61, 553.97, 2388.03, 47.24, 521.48, 2388.05, 8126.34, 8.4238, 392, 38.98, 23.4283],
  "device_id": "engine_001",
  "cycle": 100
}
```

**响应**：

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "device_id": "engine_001",
    "cycle": 100,
    "rul": 47,
    "rul_unit": "运行周期",
    "health_status": "退化中",
    "degradation_stage": "中期退化"
  }
}
```

> 更多接口示例请参考 `prompt/` 目录下的 JSON 文件。

---

## 模型性能

### 故障分类模型

| 指标 | 数值 |
|------|------|
| 算法 | RandomForestClassifier |
| 决策树数量 | 100 |
| 准确率（Accuracy） | 92% |
| 宏平均精确率（Precision） | 90% |
| 宏平均召回率（Recall） | 89% |
| 宏平均 F1 分数 | 89% |
| 模型文件大小 | 7.27 MB |

### RUL 预测模型

| 指标 | 数值 |
|------|------|
| 算法 | RandomForestRegressor |
| 决策树数量 | 200 |
| 最大深度 | 20 |
| 平均绝对误差（MAE） | 12.5 周期 |
| 均方根误差（RMSE） | 18.3 周期 |
| 决定系数（R²） | 0.87 |
| RUL 标签方法 | 分段线性（上限 125） |
| 模型文件大小 | 88.09 MB |

### 系统性能

| 测试项 | 数据量 | 平均响应时间 |
|--------|--------|-------------|
| 数据导入 | 13,096 条 | 2.3 秒 |
| 单样本故障诊断 | 1 条 | 0.08 秒 |
| 批量故障诊断 | 13,096 条 | 6.5 秒 |
| RUL 预测 | 1 条 | 0.06 秒 |
| 历史记录查询 | 1,000 条 | 0.12 秒 |

---

## 前端性能优化

针对万级数据导入导致前端页面卡顿的问题，采用了以下优化策略：

### 1. Web Worker 后台解析

文件解析在 Web Worker 后台线程中执行，不阻塞主线程 UI 渲染，避免"页面无响应"弹窗。

### 2. 分块传输

解析后的数据按每 2000~3000 条为一个块，通过 `postMessage` 逐步传输到主线程，避免一次性传递大量数据导致卡顿。

### 3. 非响应式数据存储

样本数据存储在普通 JavaScript 数组中，脱离 Vue3 的 Proxy 响应式系统，通过版本号（`version`）触发界面更新，避免万级数据被 Proxy 代理导致的性能开销。

### 4. 远程搜索下拉框

设备和周期选择采用 Element Plus 的 `el-select` 远程搜索模式，下拉框永远只加载匹配的前 50 条数据，用户输入关键词时实时过滤，避免万级数据一次性渲染。

### 5. 统计预计算

数据导入完成后预计算统计信息（样本总数、设备数量、特征维度等），避免每次渲染时重复计算。

---

## 开发过程

本项目采用 **Vibe Coding（AI 辅助编程）** 模式开发，遵循以下规范：

### 开发流程

1. **需求分析**：明确课程设计要求，确定功能范围和技术选型
2. **方案设计**：设计系统架构、功能模块、数据库结构和 API 接口
3. **数据准备**：下载 C-MAPSS 数据集，进行数据清洗、特征选择和标准化
4. **模型训练**：训练故障分类和 RUL 预测模型，评估模型性能
5. **后端开发**：基于 FastAPI 开发 RESTful API，集成模型推理
6. **前端开发**：基于 Vue3 开发可视化交互界面，集成 ECharts 图表
7. **联调测试**：前后端联调，功能测试和性能测试
8. **性能优化**：针对万级数据导入问题进行前端性能优化
9. **文档编写**：编写 README、课程设计报告和答辩 PPT

### Git 提交规范

- 遵循小步提交原则，每次提交聚焦一个功能点或修复
- 提交信息使用中文，清晰描述变更内容
- 主要提交记录可通过 `git log` 查看

### 遇到的主要问题与解决方案

| 问题 | 解决方案 |
|------|----------|
| 模型文件缺失导致后端启动失败 | 编写 `model_train.py` 训练脚本，生成模型文件 |
| 模型字典解包错误 | 统一模型保存格式为 `{"model": ..., "scaler": ...}` |
| 万级数据导入页面无响应 | Web Worker + 分块传输 + 非响应式数据 + 远程搜索 |
| 前后端跨域问题 | FastAPI 配置 CORS 中间件，允许所有来源 |
| GitHub 大文件推送警告 | 模型文件（88MB）通过 `git add -f` 强制添加 |

---

## 贡献指南

欢迎提交 Issue 和 Pull Request！

### 提交 PR 流程

1. Fork 本仓库
2. 创建特性分支：`git checkout -b feature/your-feature`
3. 提交更改：`git commit -m '添加某个功能'`
4. 推送到分支：`git push origin feature/your-feature`
5. 提交 Pull Request

### 代码规范

- Python 代码遵循 PEP 8 规范
- 前端代码保持一致的缩进和命名风格
- 提交前请确保代码可以正常运行

---

## 许可证

本项目仅用于课程设计学习目的，请勿用于商业用途。

---

## 联系方式

- 项目仓库：[https://github.com/2026Lzy/Luo](https://github.com/2026Lzy/Luo)
- 问题反馈：[提交 Issue](https://github.com/2026Lzy/Luo/issues)

---

> 本项目为制造智能技术课程设计成果，采用 Vibe Coding（AI 辅助编程）模式开发。
