# RadarAware — 毫米波雷达驾驶员疲劳监测系统

## 项目概述

RadarAware 是一个融合**毫米波雷达生命体征检测**、**计算机视觉疲劳分析**与**智能语音交互**的驾驶员状态监测系统。系统通过多传感器融合，实时计算驾驶员的心率、呼吸频率和疲劳评分，在检测到疲劳状态时主动触发语音提醒与驾驶建议，并将数据上传至远程调度中心。

## 系统架构

`
                    RadarAware
       ┌───────────────────────────────┐
       │     main.sh 系统启动脚本       │
       │  (交互菜单 / 命令行参数)        │
       └───────┬───────┬───────┬───────┘
               │       │       │
       ┌───────┘       │       └───────┐
       ▼               ▼               ▼
 ┌───────────┐ ┌───────────────┐ ┌─────────────┐
 │Radar_Real │ │Version_Detected│ │Agent_Workflow││雷达_真实 │ │版本_检测│ │智能体_工作流│
 │ 雷达信号   │ │  视觉疲劳检测  │ │ 语音交互代理 │
 │ 处理模块   │ │  (RK3588 NPU) │ │(Coze 工作流) │
 └─────┬─────┘ └───────┬───────┘ └──────┬──────┘
       │               │                │
       └───────┬───────┘                │
               │                        │
               ▼                        ▼
        ┌───────────────┐        ┌──────────────┐
        │Driver_Monitor │        │  远程服务器   │
        │  前端界面      │ ◄───── │39.108.209.209│
        │ localhost:3001│        └──────────────┘│ 本地主机：3001│ └──────────────┘│ 本地主机：3001│ └──────────────┘│ 本地主机：3001│ └──────────────┘│ 本地主机：3001│ └──────────────┘│ 本地主机：3001│ └──────────────┘│ 本地主机：3001│ └──────────────┘│ 本地主机：3001│ └──────────────┘│ 本地主机：3001│ └──────────────┘│ 本地主机：3001│ └──────────────┘│ 本地主机：3001│ └──────────────┘│ 本地主机：3001│ └──────────────┘│ 本地主机：3001│ └──────────────┘│ 本地主机：3001│ └──────────────┘│ 本地主机：3001│ └──────────────┘│ 本地主机：3001│ └──────────────┘
        └───────────────┘
## 模块说明

| 模块             | 路径                                    | 功能                                                                      |
| ---------------- | --------------------------------------- | ------------------------------------------------------------------------- |
| Driver_Monitor   | [Driver_Monitor/](./Driver_Monitor)     | 车载前端界面（1024×600 触控屏优化），提供登录、聊天、控制台、个人资料管理 ||驾驶员监测| [驾驶员监测/](./驾驶员监测)     |车载前端界面（1024×600 触控屏优化），提供登录、聊天、控制台、个人资料管理|
| Radar_Real       | [Radar_Real/](./Radar_Real)             | TI IWR6843AOP 毫米波雷达信号处理，实时提取心率与呼吸频率                  |
| Version_Detected | [Version_Detected/](./Version_Detected) | RK3588 NPU 加速的视觉疲劳检测（人脸检测 + 眼部状态 + 哈欠识别）           ||检测到的版本| [检测到的版本/](./检测到的版本) |RK3588 NPU 加速的视觉疲劳检测（人脸检测 + 眼部状态 + 哈欠识别）|
| Agent_Workflow   | [Agent_Workflow/](./Agent_Workflow)     | 语音交互代理，VAD 检测人声，调用 Coze 工作流进行疲劳提醒与驾驶建议        |

## 硬件平台

- **TI IWR6843AOP** 毫米波雷达 + **DCA1000** 数据采集卡
- **RK3588** 开发板（如 FriendlyElec NanoPC-T6 / Radxa Rock 5B）
- 车载显示终端（1024×600 触控屏）
- USB 或 MIPI 摄像头（支持 V4L2）
- 麦克风与扬声器

## 软件依赖

- Node.js（Driver_Monitor 后端）
- Python 3.8+（NumPy / SciPy / Matplotlib / statsmodels / vmdpy / OpenCV / PyAudio）
- RKNN Toolkit Lite2（RK3588 NPU 推理）
- conda 环境（推荐环境名：radar）

## 快速开始

`Bash
# 克隆项目
git clone <repo_url>
cd RadarAware

# 赋予执行权限
chmod +x main.sh

# 查看帮助
./main.sh help

# 交互式菜单启动
./main.sh

# 或直接以命令行模式启动全部模块
./main.sh --integrated
`

## main.sh 启动脚本

main.sh 是系统的统一启动脚本，支持**交互式菜单**和**命令行参数**两种模式。

### 交互式菜单

`Bash
./main.sh
`

菜单选项：

| 选项              | 说明                                       |
| ----------------- | ------------------------------------------ |
| 1) 雷达数据采集   | 启动雷达原始数据采集（radar_capture.py）   |
| 2) 雷达采集与处理 | 新窗口采集 + 前台运行心率/呼吸处理         |
| 3) 视觉疲劳检测   | 启动 RK3588 NPU 视觉检测                   |
| 4) 语音代理模块   | 启动 Agent_Workflow VAD 录音与 Coze 工作流 |
| 5) 集成模式       | 同时启动全部模块（独立窗口）               |
| 6) 系统运行状态   | 查看各模块运行状态                         |
| 7) 停止所有模块   | 发送 SIGINT 安全停止所有进程               |
| 8) 退出           | 清理并退出                                 |

### 命令行模式

`Bash
./main.sh capture      # 雷达数据采集
./main.sh processor    # 雷达数据处理（心率/呼吸）
./main.sh vision       # 视觉疲劳检测
./main.sh agent        # 语音代理
./main.sh integrated   # 集成模式（全部模块）
./main.sh status       # 查看运行状态
./main.sh stop         # 停止所有模块
`

### 启动流程

`
main.sh
  ├── 环境检查（conda、模块文件完整性）
  ├── conda 环境激活（默认：radar）
  ├── 各模块启动（独立终端窗口或后台进程）
  │   ├── Radar_Real:       radar_capture.py + main.py
  │   ├── Version_Detected: main.py
  │   ├── Agent_Workflow:   main.py
  │   └── Driver_Monitor:   node src/server.js
  ├── chromium-browser 打开前端页面 localhost:3001
  └── Ctrl+C 安全停止所有进程（SIGINT → SIGTERM）
`

### 日志与输出

- 日志文件位于 logs/ 目录，文件名格式：{模块}_{时间戳}.log
- 各模块在独立终端窗口中运行，可单独 Ctrl+C 停止
- 前端地址：http://localhost:3001

## 各模块详细文档

- [Driver_Monitor 模块](./Driver_Monitor/README.md)- [驾驶员监测模块](./Driver_Monitor/README.md)
- [Radar_Real 模块](./Radar_Real/README.md)- [雷达感知模块](./Radar_Real/README.md)
- [Version_Detected 模块](./Version_Detected/README.md)
- [Agent_Workflow 模块](./Agent_Workflow/README.md)
