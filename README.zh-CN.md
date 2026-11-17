# Heretic 中文说明

> 本仓库是 [p-e-w/heretic](https://github.com/p-e-w/heretic) 的 **fork**（只复制了 `master` 分支，代码与上游一致），额外添加这份中文说明，便于本地阅读与实验。
> 上游当前版本：**v2.1.0**（2026-11）。

## 一句话介绍

Heretic 是一个**免训练（training-free）**的大语言模型修改工具：不重新训练模型，而是用数学方法直接改动权重，让模型的某些行为特征发生变化。默认动作是**去除拒答 / 解除内容审查**，也可以改成抑制套话、降低谄媚、调节情绪、缩短回答等。

## 能做什么

| 目标 | 配置文件 |
| --- | --- |
| 去除拒答（默认） | `config.default.toml` |
| 抑制套话 | `config.noslop.toml` |
| 降低谄媚 | `config.nosyco.toml` |
| 减少幽默 | `config.nohumor.toml` |
| 调节情绪倾向 | `config.emotions.toml` |
| 缩短回答 | `config.terse.toml` |

## 环境要求与安装

- Python 3.10+
- PyTorch 2.6+（按显卡安装对应版本）
- 显存不够时，可在配置里设 `quantization = "bnb_4bit"`，运行时做 4bit 量化

```sh
pip install -U heretic-llm
heretic Qwen/Qwen3-4B-Instruct-2507
```

把模型名换成你要处理的模型即可。参考速度：RTX 3090 + 默认配置，处理 Qwen3-4B 约 30–40 分钟。

配置：程序默认读取当前目录下的 `config.toml`，也可用环境变量 `HERETIC_CONFIG_FILE` 指定路径。注意 **1.x 的配置与 2.x 不兼容**，旧配置需按官方迁移文档调整。

## 工作原理

1. **修改器（modifier）**：默认是作者自研的 **ARA（Arbitrary-Rank Ablation，任意秩消融）**；也可切换为经典 **abliteration（方向消融）**，含 projected / norm-preserving 等变体。
2. **打分器（scorer）**：默认用「拒答关键词命中率」和「KL 散度」（后者约束模型不要跑偏太远）。打分器是插件，可自行编写，用来把任意可量化的行为特征当作优化目标。
3. **优化器**：用 Optuna 的 TPE 算法跑若干轮试验，自动搜出「目标分最优、同时最不伤原模型」的参数组合。

因此它的能力边界不是一个固定的「去审查」功能，而是「把某个可度量的行为按你的偏好推过去」。

## 产物与可复现性

处理完成后可选择：保存模型、上传到 Hugging Face、直接对话测试、跑基准测试。

若上传时附带可复现信息，模型仓库里会有一个 `reproduce` 目录与 `reproduce.json` 清单（约 9KB），别人执行

```sh
heretic --reproduce path/to/reproduce.json
```

即可在几分钟内复现出字节级一致的模型，不必重跑数小时。这些清单的公开集合叫 **Heretic Grimoire**，也可以用 `--collect-reproducibles` 批量拉取到本地。

## 注意事项

- **合规风险**：本项目默认用途就是移除模型的安全对齐，生成的模型可能输出有害内容；对外提供服务前请自行评估法律与上游模型许可（如 Llama 系列的使用政策）的约束。
- **许可**：上游采用 **AGPL-3.0**，内部或商业使用时注意其传染性条款。
- **能力不变**：它只改行为倾向，不增加知识，模型自身水平不会因此提升。
- **同步 fork**：本 fork 在 `master` 上多了一次提交（即本文件），上游更新后 GitHub 的「Sync fork」可能出现分支分叉提示；若要干净同步，可把本文件维护在单独分支。

## 相关链接

- 上游仓库：https://github.com/p-e-w/heretic
- 项目主页：https://heretic-project.org
- 上游许可证：AGPL-3.0

---

本中文说明为本地维护，仅作阅读辅助；以英文 README 与官方文档为准。
