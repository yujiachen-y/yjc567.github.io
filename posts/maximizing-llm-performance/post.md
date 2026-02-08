

> 注：这里的”性能“原文为”performance”，指 LLM 回答问题的准确率，而不是运行时的计算资源消耗性能。

原视频来自 OpenAI

[https://www.youtube.com/watch?v=ahnGLM-RC1Y](https://www.youtube.com/watch?v=ahnGLM-RC1Y)

# 优化 LLM 性能的难点

- 从各种噪声中找到信号，辨认出问题并不容易。
- LLM 的性能评估抽象且难量化。
- 不知道从何入手解决问题。

# 优化路径

LLM 的优化路径

- 上下文优化，让模型知道更多。
- LLM 优化，改变模型的行为和方法。

![](attachments/Pasted%20image%2020250502123545.png)

一般的演化方向

1. Prompt engineering
2. RAG，类比给 LLM 加上短期记忆，针对具体问题提供具体信息。
3. Fine-tune model，类比给 LLM 加上长期记忆，让模型持续地遵循某种行为模式或输出结构。

拿考试举例子，Fine-tuning 是闭卷考，你需要记住所有知识才能参加考试；RAG 是开卷考，你带着各种参考资料进考场。

![](attachments/Pasted%20image%2020250502123557.png)

# Prompt Engineering

先用简单的提示词工程尝试解决问题，找到评估基准，判断性能瓶颈是上下文还是模型的行为模式。

当确定基准后，可以在 prompt 里添加 few-shot 示例，看是否可以带来性能提升。

## 优点

- 尽快地测试并获得反馈。
- 结合性能评估，这确定了性能优化的基准线。

## 缺点

- prompt 中引入了新信息。
- 被上下文窗口限制。
- token 使用效率低。

## 最佳实践

- 编写清晰的指令。
- 把复杂任务拆分成简单的子任务。
- 让 LLM 有时间思考，例如 CoT，ReAct。
- 用控制变量法系统性地测试改动控制变量法。

## 后续步骤

- 提供参考文本。
- 接入外部工具。

# RAG

先根据问题找到相关的上下文，再要求 LLM 回答。

## 优点

- 可以不断向 LLM 提供最新的信息。
- 通过控制参考文本降低幻觉。

## 缺点

- 提供给 LLM 的信息仅限上下文片段，无法让 LLM 拥有更宽泛的领域知识。
- 不擅长让 LLM 按照规定格式或特定语言输出结果。
- 不利于降低 token 使用率。

## 进阶技巧

- 用 cosine 计算向量相似度。
- HyDE (Hypothetical Document Embedding)，让模型先回答，用回答去搜索上下文。
- Fine-tuning embedding 模型。
- 分片后再进行 embedding。
- 重排，搜索到上下文后，重新计算和问题的相似程度，或施加一些人工规则，重新排序和剔除上下文。
- 分类，让模型先判断问题相关的上下文有可能在哪些数据集中找到，之后再进行搜索。
- 并行计算，把问题拆分成多个子问题，独立计算后合并答案。

## 评估

RAG 实质上给 LLM 引入了一个新的环节：检索，这也可能成为一个瓶颈。检索结果的质量很重要，如果一些无关、或者低质量的上下文被检索到，LLM 会给出和幻觉无差别的低质量回答。

[https://github.com/explodinggradients/ragas](https://github.com/explodinggradients/ragas)

这里介绍了 Ragas 用来评测 RAG 的表现，评测分为两大类四个维度

- 生成，LLM 回答的质量。
    - Faithfulness，生成回答的真实准确性。
    - Answer relevancy，生成回答和问题的相关性。
- 检索，被检索文本和问题的相关性。
    - Context precision，被检索内容的信噪比。
    - Context recall，能否检索出所有和问题相关的内容。

![](attachments/Pasted%20image%2020250502123619.png)

# Fine-tuning

继续在更小、特定领域的数据集上训练，优化特定任务的表现和效率。

- 提高模型在特定任务上的表现。
    - 不需要用 prompt 来规范模型的表现；和 few-shot learning 相比，你可以让模型充分学习相关的数据。
- 提高模型的效率。
    - 节省更多的上下文窗口，LLM 处理更快也更节省 token。
    - 可以让小模型在特定任务上达到更多参数模型的表现，而小模型的费用和延时更优。

## 优点

- 强化模型中已有的知识。
- 自定义回答的结构或语气。
- 让模型学会复杂的指令，避开复杂的 prompt engineering。

## 缺点

- 如果 prompt engineering 无法提高模型的表现，那么 fine-tuning 也不行。
- 无法向基础模型中添加新知识。
- 无法适应新使用场景。

## 例子

Canva 使用 fine-tuning 来规范 LLM 的输出格式。

一个博客作者使用 Slack 聊天记录来 fine-tuning LLM，期望 LLM 学会他的语气，但 LLM 实际上学会了他消极怠工的态度，因此高质量的 fine-tuning 数据很重要。

## 步骤

1. 准备数据：收集、验证、格式化数据。
2. 训练：选择超参数、损失函数，注意 LLM 的损失函数是 next token prediction 任务的代理，但这和 LLM 负责的下游任务不一定有相关性，比如在代码生成中，你有很多方式来解决一个代码问题，而不需要生成的代码和标准答案完全匹配。
3. 评估：人工评估、LLM 评估。
4. 推理

![](attachments/Pasted%20image%2020250502123638.png)

## 最佳实践

- 先从 prompt engineering 和 few-shot learning 开始，确认 LLM 适合你的使用场景。
- 确认性能基准，确保 fine-tuning 后的 LLM 表现可衡量可比较。
- 从小数据集开始，注重数据质量，LLM 的训练过程已经解决了数据量的问题，fine-tuning 要着重注意数据质量。

## 引用本文

APA：
Yu, J. (2023年11月26日). 最大化 LLM 性能的技术概览. Jiachen Yu. https://www.yujiachen.com/maximizing-llm-performance/

BibTeX：
```bibtex
@online{yu2023maximizingllmperformance,
  author = {Yu, Jiachen},
  title = {最大化 LLM 性能的技术概览},
  year = {2023},
  publisher = {Jiachen Yu},
  url = {https://www.yujiachen.com/maximizing-llm-performance/},
  urldate = {2026-02-08},
}
```
