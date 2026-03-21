# LM Studio Qwen3-Coder-Next 调优记录

## 已确认信息

- 模型变体：`qwen/qwen3-coder-next@q4_k_m`
- 当前硬件：`Ryzen AI Max+ 395`
- LM Studio 当前已成功加载的默认上下文：`8192`
- 真实旧配置中曾使用：
  - `contextLength = 262144`
  - `numExperts = 10`
  - `flashAttention = true`
  - `offloadRatio = 1`

## 已应用的默认参数

已写入 LM Studio per-model defaults：

- 文件：`C:\Users\ync\.lmstudio\.internal\user-concrete-model-default-config\qwen\qwen3-coder-next.json`
- 参数：
  - `contextLength = 8192`
  - `flashAttention = true`
  - `evalBatchSize = 256`
  - `offloadKVCacheToGpu = false`
  - `numExperts = 10`
  - `offloadRatio = 1`

## 为什么这样配

### 1. contextLength = 8192

你的主要卡顿来自 prefill，也就是 Prompt Processing。

把上下文从 `262144` 降到 `8192` 后：

- KV Cache 压力显著下降
- 统一内存带宽竞争明显减轻
- 首字前的 GPU 抢占会收敛很多
- Windows 桌面卡死风险明显下降

资源估算：

- `8192` -> `46.81 GiB`
- `16384` -> `47.11 GiB`
- `32768` -> `47.70 GiB`
- `262144` -> `55.94 GiB`

### 2. flashAttention = true

这是长上下文和 MoE 模型下最值得保留的优化项之一。

保留它的原因：

- 能降低注意力计算压力
- 对 KV 和长 prompt 更友好
- 在你的 UMA 架构上通常比关闭更稳

### 3. evalBatchSize = 256

这是这次最终选定的平衡点。

建议理解为：

- `128`：更稳，Prompt Processing 更轻，首字延迟更容易收敛，但吞吐通常更低
- `256`：速度与卡顿之间最均衡，适合作为日常默认值
- `512`：prefill 速度可能更快，但更容易把 GPU 和共享带宽压满，导致桌面更卡

默认引擎里的常见默认值是 `512`，但对你的机器来说偏激进。

### 4. offloadKVCacheToGpu = false

在独显机器上，这个选项更容易带来收益；
但在你的统一内存架构上，GPU 本身也承担桌面显示，KV 再压到 GPU 会更容易出现：

- 鼠标明显延迟
- 窗口拖动掉帧
- 首字前系统近似假死

因此默认改为 `false` 更适合你的机器。

### 5. numExperts = 10

这是模型原本的默认专家数，先不降。

原因：

- 你当前的主要问题是系统交互卡顿，不是模型质量不够
- 先通过 `contextLength / evalBatchSize / KV offload` 降负载更划算
- 若以后仍想继续压首字时间，再考虑把专家数往下调做单独试验

## 建议档位

### 日常编码默认

- `contextLength = 8192`
- `flashAttention = true`
- `evalBatchSize = 256`
- `offloadKVCacheToGpu = false`
- `numExperts = 10`

### 更稳更顺滑

- `contextLength = 8192`
- `flashAttention = true`
- `evalBatchSize = 128`
- `offloadKVCacheToGpu = false`

适合：

- 边开 IDE 边跑 LM Studio
- 对 Windows 交互流畅度要求高
- 希望减少 Prompt Processing 阶段卡顿

### 更长上下文

- `contextLength = 16384`
- `flashAttention = true`
- `evalBatchSize = 256`
- `offloadKVCacheToGpu = false`

适合：

- 一次塞更多文件
- 做中等规模代码审阅

### 非必要不推荐的极限档

- `contextLength = 32768`
- `flashAttention = true`
- `evalBatchSize = 128`
- `offloadKVCacheToGpu = false`

只在确实需要长上下文时使用。

## 观测到的旧配置真实耗时

旧配置下，有一次真实请求日志显示：

- prompt tokens: `6654`
- prompt eval time: `16719.90 ms`
- generation time: `7126.76 ms / 235 tokens`
- total time: `23846.66 ms`

这说明旧配置的主要瓶颈确实是 prefill，而不是 decode。

## 如果你还想继续压卡顿

下一步优先级建议：

1. 把 `evalBatchSize` 再降到 `128`
2. 继续保持 `offloadKVCacheToGpu = false`
3. 上下文不要超过 `16384`
4. 如果仍不满意，再考虑更低量化或更小模型
