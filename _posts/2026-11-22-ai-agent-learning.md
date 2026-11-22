---
layout: post
title: "AI Agent 入门：从工具调用到第一个可测试的循环"
date: 2026-11-22 12:00:00 +0800
description: "用测试用例查询做一个最小的 AI Agent，理解工具调用、运行边界和评估方法。"
tags: [AI Agent, Python, 软件测试]
---

假设你想查「结算模块有哪些测试用例」。模型可以根据经验列出优惠券、运费和退款场景，但要回答项目里实际覆盖了什么，它需要查测试用例库。查到结果后，它还要判断是继续查询、请你补充信息，还是已经可以回答。学习 AI Agent，可以从这个小任务开始。

![AI Agent 的控制台连接查询工具和测试结果，工具结果沿回路返回控制台](/images/blog/ai-agent-learning.jpg)

## 什么时候需要 Agent

先看谁决定下一步。如果需求是「根据模块名查用例，然后总结」，用固定工作流就够了。程序先查，再把查询结果交给模型整理。如果用户只说「优惠券改了，帮我看看要回归什么」，模型可能需要先查询优惠券模块，再根据结果决定是否追查结算相关用例。后者适合练习由模型选择下一步的 Agent 循环。

循环可以拆成四步。

1. 把用户目标、可用工具和此前的查询结果交给模型。
2. 模型选择调用工具、给出答案，或提出一个澄清问题。
3. 应用校验工具名和参数，执行查询，再把返回值记为一条观察结果。
4. 把观察结果送回模型，直到任务结束或达到调用上限。

以查测试用例为例，「结算」是输入，「CHECKOUT-01 和 CHECKOUT-02」是可核验的查询结果。把用例编号保留到最终答案中，读者就能回到原记录核对测试范围。

## 用 Python 跑通一个最小循环

下面的示例可以保存为 `agent_demo.py`，用 `python3 agent_demo.py` 直接运行。`demo_decide` 是一个确定性的模型替身，方便在接入模型 API 之前验证调用、结果回填和退出三件事。用例库由两条本地样本组成。

```python
def find_test_cases(module):
    catalog = {
        "结算": [
            "CHECKOUT-01 优惠券抵扣后重新计算应付金额",
            "CHECKOUT-02 撤销优惠券后恢复原金额",
        ]
    }
    return catalog.get(module, [])


def run_agent(goal, decide, max_tool_calls=3):
    observations = []
    tools = {"find_test_cases": find_test_cases}

    while True:
        action = decide(goal, tuple(observations))
        if action["type"] in ("answer", "clarify"):
            return action

        if len(observations) >= max_tool_calls:
            return {"type": "stopped", "text": "已达到查询上限。"}

        tool = tools.get(action.get("tool"))
        module = action.get("module")
        if action["type"] != "tool" or tool is None:
            return {"type": "stopped", "text": "工具调用无效。"}
        if not isinstance(module, str) or not module.strip():
            return {"type": "clarify", "text": "请提供有效的模块名。"}

        module = module.strip()
        observations.append({"module": module, "cases": tool(module)})


def demo_decide(goal, observations):
    if not observations:
        if "结算" not in goal:
            return {"type": "clarify", "text": "你想查询哪个模块？"}
        return {"type": "tool", "tool": "find_test_cases", "module": "结算"}

    cases = observations[-1]["cases"]
    text = "\n".join(cases) if cases else "用例库里没有查到匹配记录。"
    return {"type": "answer", "text": text}


if __name__ == "__main__":
    result = run_agent("结算模块有哪些测试用例？", demo_decide)
    print(result["text"])
```

运行后会得到两条用例。

```text
CHECKOUT-01 优惠券抵扣后重新计算应付金额
CHECKOUT-02 撤销优惠券后恢复原金额
```

接入模型时，把 `demo_decide` 换成一次模型调用，让模型返回结构化的动作。上面的示例只校验了工具是否已登记和模块名是否有效。真正接入 API 时，还要先按 schema 校验完整动作，再执行工具。查询超时、查询成功但结果为空，也要以不同的状态回填，模型才能决定重试、提问还是结束。

## 先测工具调用，再测回答

只看最终回答容易漏掉问题。即使总结读起来通顺，它也可能查错了模块，或引用了不存在的编号。可以从下面几种输入开始，每次同时检查工具名、参数、调用次数和最终用例编号。

| 场景 | 期望结果 |
| --- | --- |
| 查询已知的结算模块 | 调用一次查询工具，返回样本里的两个编号 |
| 用户没有给出模块 | 提出澄清问题，查询次数为零 |
| 查询成功但无匹配记录 | 明确说明未查到用例，引用编号为空 |
| 模型反复请求同一个工具 | 达到次数上限时停止，返回明确状态 |

在接入真实用例库后，可以记录每轮的工具名、参数、返回状态和耗时。这样发现漏查时，可以定位是模型选错了查询条件、工具没找到记录，还是模型整理结果时丢了条目。等这些路径能稳定通过，再增加第二个只读工具，例如查询近期需求变更，用它判断哪些用例值得优先回归。

延伸阅读：[Anthropic《Building effective agents》](https://www.anthropic.com/research/building-effective-agents)。
