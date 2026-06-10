# 给 AI Agent 加一层回归测试：AgentProbe

普通软件改完代码会跑测试，但很多 AI Agent 改完 prompt、换模型或升级 SDK 后，仍然只能靠手动试一遍。

问题是 agent 的回归通常不是直接报错。接口还能返回，CI 还是绿的，但行为可能已经变了：本来应该先 search 再 summarize，现在直接开始编；本来应该输出 JSON，现在混进自然语言；本来不该调用危险工具，planner 却多走了一步。

这些问题需要一层更贴近 agent 行为的测试。

所以我做了 [AgentProbe](https://github.com/he-yufeng/AgentProbe)，一个给 AI Agent 做回归测试的 pytest 插件。

```bash
pip install agentprobe
```

它的核心思路类似 snapshot testing：把 agent 的一次输出存成 baseline，下次测试时把新输出和旧输出对比。对于确定性输出，可以精确匹配；对于自然语言输出，可以使用语义相似度比较。

```python
from agentprobe import snapshot

@snapshot("summarize_article")
def test_summarize():
    result = my_agent.summarize("The quick brown fox jumps over the lazy dog.")
    return result
```

第一次运行会生成 `.agentprobe/snapshots/summarize_article.json`。之后如果输出漂移，测试会失败，并在 CI 日志中显示 diff。

AgentProbe 还提供了几类对 agent 更实用的测试能力：

- `MockLLM`：不用真实 API 测试 planner、parser、retry 和工具调度逻辑。
- Tool call assertions：检查工具是否被调用、参数是否正确、顺序是否符合预期。
- Pydantic schema validation：校验结构化输出，避免 JSON 或字段结构悄悄坏掉。

这个项目不是大型 eval 平台。它更像一层轻量安全网，适合本地 agent、RAG agent、coding agent 和工具调用 agent 的关键路径测试。

我自己的判断是，未来写 agent 不能只靠“手动跑一下看起来还行”。只要 agent 有工具调用、多轮状态或结构化输出，就应该有一层回归测试，至少能在 prompt、模型和依赖变化后发现行为漂移。

项目地址：[https://github.com/he-yufeng/AgentProbe](https://github.com/he-yufeng/AgentProbe)

