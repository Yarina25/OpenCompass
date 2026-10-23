# OpenCompass 评测部署包（API 模型可直接跑）

一套从环境安装 → 配置模型 → 运行评测 → 看结果的可直接落地脚本。只评测 API 模型（DeepSeek / 通义 / 智谱 / OpenAI / 火山 / Ollama 本地）时，**不需要 GPU**。

## 目录结构

```
opencompass_eval/
├── README.md                         # 本说明
├── env.example                       # 环境变量样例（KEY / base_url / 模型名）
├── configs/
│   ├── model_openai_compatible.py    # 单模型配置（环境变量驱动，默认用这个）
│   ├── model_ab_compare.py            # A/B 双模型并排对比配置（可选）
│   └── eval_api_demo.py              # 全配置文件模式模板（备选）
└── scripts/
    ├── 01_install.sh                 # 一键装环境
    └── run_eval.sh                   # debug/run/resume/viz/list 运行封装
```

---

## 一、快速开始（5 步）

### 1. 装环境（约 5–15 分钟，取决于网络）

```bash
# 默认装到 ~/opencompass（源码）和 ~/oc_venv（虚拟环境）
bash opencompass_eval/scripts/01_install.sh
```

> 没有 conda 也能跑，脚本用 Python venv。推荐 Python 3.10 / 3.11；
> 如需指定：`PYTHON_BIN=python3.11 bash opencompass_eval/scripts/01_install.sh`

### 2. 填密钥

```bash
cp opencompass_eval/env.example ~/.oc_env
# 编辑 ~/.oc_env：填 LLM_API_KEY，选好 LLM_API_BASE 和 LLM_MODEL_NAME
source ~/.oc_env
```

### 3. 冒烟测试（官方 demo 数据集，串行打印，先验证鉴权通不通）

```bash
source ~/oc_venv/bin/activate          # 若安装脚本已写 oc.env 也可 source ~/oc_venv/oc.env
bash opencompass_eval/scripts/run_eval.sh debug
```

看到模型正常返回、无 401/404，说明配置 OK。

### 4. 选正式基准并正式评测

先查可用数据集名（注册名带版本哈希，必须以本机查询为准）：

```bash
cd ~/opencompass
python tools/list_configs.py gsm8k      # 查 gsm8k 相关
python tools/list_configs.py mmlu      # 查 mmlu 相关
```

把查到的名字填入环境变量后正式跑：

```bash
export DATASETS="上一步查到的数据集名"
bash opencompass_eval/scripts/run_eval.sh run
```

### 5. 看结果

终端直接打印得分表；文件在 `outputs/<时间戳>/` 下：

```
outputs/<时间戳>/
├── predictions/   # 模型原始输出（逐条，可人工复核）
├── results/       # 各任务得分
└── summary/      # 汇总表（csv / txt）
```

---

## 二、厂商 base_url 对照表（填到 LLM_API_BASE）

| 厂商 | LLM_API_BASE | LLM_MODEL_NAME 示例 |
|---|---|---|
| DeepSeek | `https://api.deepseek.com` | `deepseek-chat` / `deepseek-reasoner` |
| 通义千问 | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `qwen-turbo` / `qwen-plus` / `qwen-max` |
| 智谱 GLM | `https://open.bigmodel.cn/api/paas/v4/` | `glm-4-flash` / `glm-4-plus` |
| OpenAI | `https://api.openai.com/v1` | `gpt-4o` / `gpt-4o-mini` |
| 火山方舟 | `https://ark.cn-beijing.volces.com/api/v3` | `ep-xxxxxxxx`（接入点 ID，不是模型名） |
| Ollama 本地 | `http://localhost:11434/v1` | `qwen:7b`（KEY 任意填） |

> OpenAISDK 走标准 Chat Completions 协议，任何兼容该协议的网关/自建服务都能接。

---

## 三、常用基准方向（用 list_configs.py 搜对应目录名）

| 能力方向 | 搜索关键字 | 说明 |
|---|---|---|
| 综合知识 | `mmlu` / `mmlu_pro` / `ceval` / `cmmlu` | 多学科选择题 |
| 数学推理 | `gsm8k` / `math` | 多步计算 |
| 代码 | `humaneval` / `mbpp` | 代码生成（humaneval 需额外装依赖，见下） |
| 逻辑/常识 | `bbh` / `hellaswag` / `arc` / `winogrande` / `piqa` | 推理与常识 |
| 幻觉/事实 | `truthfulqa` | 是否编造 |

冒烟用官方 `demo_gsm8k_chat_gen`，成本最低。

---

## 四、run_eval.sh 子命令

| 命令 | 作用 |
|---|---|
| `run_eval.sh debug` | 串行 + 实时打印，排查鉴权/格式问题（默认 demo 数据集） |
| `run_eval.sh run` | 正式并行评测 |
| `run_eval.sh resume` | `-r` 复用已完成结果，只补跑失败/未完成任务（断网中断后续跑） |
| `run_eval.sh viz` | `--mode viz` 不调用模型，只对已有结果重新汇总出表 |
| `run_eval.sh list <关键字>` | 模糊查询可用模型/数据集配置名 |

常用覆盖变量：`MODEL_CFG`（换模型/对比配置）、`DATASETS`（空格分隔多个）、`WORK`（输出目录）、`MAX_WORKERS`（并行任务数）、`OC_DIR`（源码目录）。

### A/B 双模型对比

```bash
# 在 ~/.oc_env 配好 LLM_A_* 和 LLM_B_* 后：
MODEL_CFG=opencompass_eval/configs/model_ab_compare.py bash opencompass_eval/scripts/run_eval.sh run
```

---

## 五、常见报错排查

| 现象 | 原因 / 处理 |
|---|---|
| `401 Unauthorized` | KEY 错/过期/余额不足；检查 `LLM_API_KEY` 是否 export 且无多余空格 |
| `404 Not Found` | `LLM_API_BASE` 写错，末尾 `/v1` 不要乱加；厂商地址不能混用 |
| `429 Too Many Requests` | 触发限流，调小 `query_per_second`（模型配置里）和 `MAX_WORKERS` |
| 连接超时 | 网络访问不到该 API；国外模型需自备网络；本地 Ollama 先确认已启动 |
| 找不到数据集名 | 注册名带版本哈希，必须 `python tools/list_configs.py <关键字>` 查本机真实名字 |
| humaneval 跑不了 | 需单独克隆安装 human-eval/evalplus，见官方「数据集准备」说明 |
| 结果两次不一致 | 评测必须固定 `temperature=0`（配置已默认），不要开采样 |

---

## 六、补充说明

- **可复现**：模型配置已默认 `temperature=0`、`top_p=1.0`，横向对比时各模型参数保持一致。
- **数据集下载**：默认 `DATASET_SOURCE=ModelScope`（国内自动下载）；删掉该环境变量回退 HuggingFace。
- **不想克隆源码（pip 快速版）**：`pip install -U "opencompass[api]"` 后用 CLI `opencompass --models <配置> --datasets <数据集>`；但 `tools/list_configs.py`、`configs/`、`run.py` 仅源码版自带，排查问题仍推荐源码版。
- **成本**：正式全量基准（如完整 MMLU、MATH）调用量较大，建议先用小样本或 demo 验证，再放量。
- **安全**：不要把真实 KEY 写进配置文件或提交 git；API 评测本身不执行模型生成的代码，代码类 Pass@1 需另在沙箱跑（见此前 Pass@1 脚本）。
