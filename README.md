# OpenCompass 评测部署包（API 模型可直接跑）

一套从环境安装 → 配置模型 → 运行评测 → 看结果的可直接落地脚本。只评测 API 模型（DeepSeek / 通义 / 智谱 / OpenAI / 火山 / Ollama 本地）时，**不需要 GPU**。

## 仓库内容

| 文件 | 说明 |
|---|---|
| `opencompass_eval.zip` | 评测部署脚本包（一键装环境、配置、运行、汇总） |
| `OpenCompass评测部署手册.docx` | 图文部署手册（逐步操作） |
| `README 使用说明.md` | 详细使用说明（命令、厂商对照表、排查手册） |

## 快速开始（5 步）

1. **装环境**：`bash opencompass_eval/scripts/01_install.sh`（约 5–15 分钟，Python 3.10/3.11 推荐）
2. **填密钥**：`cp opencompass_eval/env.example ~/.oc_env`，编辑填入 `LLM_API_KEY` / `LLM_API_BASE` / `LLM_MODEL_NAME`
3. **冒烟测试**：`bash opencompass_eval/scripts/run_eval.sh debug`（官方 demo 数据集，验证鉴权）
4. **正式评测**：用 `python tools/list_configs.py <关键字>` 查数据集名后，`DATASETS="..." bash opencompass_eval/scripts/run_eval.sh run`
5. **看结果**：终端打印得分表，文件在 `outputs/<时间戳>/` 下

## 支持厂商（OpenAISDK 兼容协议）

| 厂商 | LLM_API_BASE | LLM_MODEL_NAME 示例 |
|---|---|---|
| DeepSeek | `https://api.deepseek.com` | `deepseek-chat` / `deepseek-reasoner` |
| 通义千问 | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `qwen-turbo` / `qwen-plus` / `qwen-max` |
| 智谱 GLM | `https://open.bigmodel.cn/api/paas/v4/` | `glm-4-flash` / `glm-4-plus` |
| OpenAI | `https://api.openai.com/v1` | `gpt-4o` / `gpt-4o-mini` |
| 火山方舟 | `https://ark.cn-beijing.volces.com/api/v3` | `ep-xxxxxxxx`（接入点 ID） |
| Ollama 本地 | `http://localhost:11434/v1` | `qwen:7b`（KEY 任意填） |

> 任何兼容 Chat Completions 协议的网关/自建服务都能接入。

## 常用基准方向

| 能力方向 | 搜索关键字 |
|---|---|
| 综合知识 | `mmlu` / `mmlu_pro` / `ceval` / `cmmlu` |
| 数学推理 | `gsm8k` / `math` |
| 代码 | `humaneval` / `mbpp` |
| 逻辑/常识 | `bbh` / `hellaswag` / `arc` / `winogrande` / `piqa` |
| 幻觉/事实 | `truthfulqa` |

冒烟测试用官方 `demo_gsm8k_chat_gen`，成本最低。

## 注意事项

- **可复现**：模型配置默认 `temperature=0`、`top_p=1.0`，横向对比参数一致
- **数据集下载**：默认 `DATASET_SOURCE=ModelScope`（国内自动下载）
- **安全**：不要把真实 KEY 提交到 git；API 评测不执行模型生成的代码
- **完整说明**：详见仓库内 `README 使用说明.md` 和部署手册
