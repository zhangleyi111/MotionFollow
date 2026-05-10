[README.md](https://github.com/user-attachments/files/27565593/README.md)
# MotionFollow - 动作跟跳学习系统

将任意舞蹈/健身视频转化为结构化动作序列，通过摄像头实时捕捉用户姿态，对比评分并生成练习报告。

## 环境要求

- Python 3.12+
- Node.js 18+
- npm

## 快速启动

### 1. 启动后端（终端 1）

```bash
cd hack

# 创建虚拟环境（首次）
python -m venv .venv
source .venv/bin/activate

# 安装依赖
pip install -r backend/requirements.txt

# 启动服务
python -m uvicorn backend.main:app --host 0.0.0.0 --port 8000 --reload
```

后端运行在 http://localhost:8000

### 2. 启动前端（终端 2）

```bash
cd hack/frontend

# 安装依赖（首次）
npm install

# 启动开发服务
npm run dev
```

前端运行在 http://localhost:3000

### 3. 打开浏览器

访问 http://localhost:3000

## 配置

编辑 `backend/.env`：

```
DEEPSEEK_API_KEY=你的key
DEEPSEEK_BASE_URL=https://api.deepseek.com
UPLOAD_DIR=./uploads
SEGMENT_DURATION=2.5
```

不配置 DeepSeek API Key 也能用，动作标签会用默认名称。

## 使用流程

1. 拖拽或点击上传一个动作视频（支持 `青海摇.mp4`、`向快乐出发.mp4` 等预设视频）
2. 等待 AI 自动拆解（约 3-10 秒），预设视频会直接使用预置配置
3. 查看时间轴，点击任意片段跳转预览
4. 点击「开始跟练」进入跟练模式，允许摄像头权限
5. 系统每 2.5 秒提取 20 帧与原视频对比，实时评分
6. 每段结束后显示 Perfect / Good / Miss 粒子反馈
7. 全部完成后进入报告页，可生成分享海报

## 评分机制

- 每个动作片段（2.5 秒）等间隔提取 20 帧骨架
- 用户侧同步提取 20 帧进行对比
- 评分 = 姿态匹配分 × 运动幅度因子
- 支持镜像模式（用户面对摄像头）
- 综合评定 S / A / B / C 等级

## API 接口

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | /api/upload | 上传视频 |
| POST | /api/analyze/{video_id} | 分析视频动作 |
| GET | /api/analysis/{video_id} | 获取分析结果 |
| POST | /api/score-log | 记录评分日志 |
| GET | /api/health | 健康检查 |

## 项目结构

```
hack/
├── backend/
│   ├── main.py                  # FastAPI 服务（上传、分析、评分日志）
│   ├── requirements.txt         # Python 依赖
│   ├── .env                     # 环境变量
│   ├── models/                  # MediaPipe 姿态检测模型
│   │   ├── pose_landmarker.task
│   │   └── pose_landmarker_lite.task
│   ├── configs/                 # 预设视频配置（跳过 AI 分析）
│   │   └── 向快乐出发.mp4.json
│   └── uploads/                 # 上传视频存储
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── page.tsx              # 首页：上传 + 分析
│   │   │   ├── practice/page.tsx     # 跟练页：摄像头 + 实时评分
│   │   │   └── report/page.tsx       # 报告页：成绩 + 分享海报
│   │   ├── components/
│   │   │   ├── VideoUploader.tsx      # 视频上传组件
│   │   │   └── Timeline.tsx          # 动作时间轴
│   │   └── lib/
│   │       ├── api.ts                # API 封装
│   │       └── scoring.config.ts     # 评分参数配置
│   ├── next.config.ts                # Next.js 配置（API 代理）
│   └── package.json
├── uploads/                          # 视频上传目录
├── log/                              # 评分日志
├── 青海摇.mp4                        # 示例视频
└── prd.md                            # 产品需求文档
```

## 技术栈

- **后端**: FastAPI + MediaPipe Pose + DeepSeek LLM + OpenCV
- **前端**: Next.js 16 + React 19 + Tailwind CSS 4 + TypeScript
- **姿态检测**: MediaPipe PoseLandmarker（后端视频分析 + 前端摄像头实时捕捉）
