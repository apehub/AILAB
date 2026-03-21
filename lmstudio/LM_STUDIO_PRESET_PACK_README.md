# LM Studio Preset Pack

已为 `qwen/qwen3-coder-next@q4_k_m` 生成 4 个本地 `LM Studio` 预设，文件已放入：

- `C:\Users\ync\.lmstudio\config-presets`

## 预设列表

- `Qwen3CoderNext-8K-Smooth`
  - `contextLength = 8192`
  - `evalBatchSize = 128`
  - 适合最顺滑的日常编码体验

- `Qwen3CoderNext-8K-Balanced`
  - `contextLength = 8192`
  - `evalBatchSize = 256`
  - 适合作为默认开发档

- `Qwen3CoderNext-16K-Balanced`
  - `contextLength = 16384`
  - `evalBatchSize = 256`
  - 适合 2 到 5 个文件联动分析

- `Qwen3CoderNext-32K-LongContext`
  - `contextLength = 32768`
  - `evalBatchSize = 128`
  - 仅在必须长上下文时使用

## 共同参数

所有预设都保持：

- `flashAttention = true`
- `offloadKVCacheToGpu = false`
- `numExperts = 10`
- `offloadRatio = 1`

## 使用建议

- 默认优先用 `Qwen3CoderNext-8K-Balanced`
- 如果更在意系统顺滑度，用 `Qwen3CoderNext-8K-Smooth`
- 只有确实需要更多上下文时，再切到 `16K` 或 `32K`
