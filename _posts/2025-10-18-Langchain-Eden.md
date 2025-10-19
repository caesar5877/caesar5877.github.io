prompt: 下面是字幕，如何建立agent。根据字幕，给我步骤

Section 14: Reflectin Agent
非常好，这段字幕是讲 **如何用 LangGraph 建立一个完整的 AI Agent（基于 Flow Engineering 思想）**。
下面我将把整段内容拆解成 **逐步构建流程**，并解释每一步的 **目的、关键点、和代码逻辑**。

---

## 🧭 一、总体目标

要构建一个具备「**生成（Generate）→ 反思（Reflect）→ 再生成**」循环逻辑的 Agent。

LangGraph 用「**节点（Nodes）+ 边（Edges）+ 状态（State）**」描述整个流程。
目标是构建一个工作流（Workflow），让 LLM 在既定流程中循环生成与改进输出。

---

## 🧩 二、构建步骤总览（宏观流程）

| 阶段     | 主要工作                     | 结果             |
| ------ | ------------------------ | -------------- |
| Step 1 | 定义 Chains（生成链、反思链）       | Agent 的基本逻辑块   |
| Step 2 | 定义 Nodes（节点）             | 每个节点执行一个 chain |
| Step 3 | 定义 State（状态结构）           | 保存上下文消息历史      |
| Step 4 | 构建 Graph（图结构）            | 把节点、边、条件逻辑连起来  |
| Step 5 | 定义条件函数 `should_continue` | 决定继续循环还是结束     |
| Step 6 | 编译、绘制并测试 Graph           | 输出可视化并验证流程     |

---

## 🧱 Step 1：定义 Chains（已在前一节完成）

* **Generate Chain**：生成推文（AI 输出）
* **Reflect Chain**：对生成结果进行反思、批评
* 每个 Chain 之后都会返回一个 **消息对象（BaseMessage / AIMessage）**

---

## 🧠 Step 2：定义 Nodes（节点）

每个节点对应一个 chain：

| 节点           | 调用的 Chain        | 输入                    | 输出                | 功能     |
| ------------ | ---------------- | --------------------- | ----------------- | ------ |
| **generate** | `generate_chain` | 当前状态（message history） | AI 生成的新消息         | 执行生成逻辑 |
| **reflect**  | `reflect_chain`  | 当前状态                  | 人类消息（模拟 critique） | 执行反思逻辑 |

> 生成节点的输出是 `AIMessage`，反思节点的输出是 `HumanMessage`（这是一个 prompt engineering 技巧，让模型认为批评来自人类，从而提升响应质量）。

---

## 📦 Step 3：定义 State（状态结构）

状态用于保存所有消息的上下文。
使用 `TypedDict` 声明结构化数据：

```python
class MessageGraph(TypedDict):
    messages: Annotated[List[BaseMessage], add_messages]
```

* `messages`：保存所有的历史消息（人类与 AI）；
* `add_messages`：一个 **reducer 函数**，定义如何更新状态；

  * 每次节点执行后新消息会被「append」到历史列表，而不是覆盖；
  * 这保证了对话上下文持续积累。

---

## 🧩 Step 4：创建 Graph 并添加 Nodes

创建一个 `StateGraph` 对象：

```python
workflow = StateGraph(state_schema=MessageGraph)
```

添加节点：

```python
workflow.add_node("generate", generation_node)
workflow.add_node("reflect", reflection_node)
```

设置 **入口节点（entry point）**：

```python
workflow.set_entry_point("generate")
```

> 这会在内置的 `Start` 节点与 `generate` 节点之间自动创建一条初始边。

---

## 🔀 Step 5：定义 Edges（边与条件逻辑）

### (1) 固定边（Deterministic Edge）

* `reflect → generate`：每次反思后必定进入新的生成。

### (2) 条件边（Conditional Edge）

* `generate → reflect` 或 `generate → end`：由条件函数 `should_continue` 决定。

```python
def should_continue(state: MessageGraph) -> str:
    if len(state["messages"]) >= 6:
        return "end"
    else:
        return "reflect"
```

然后在图中添加这条条件边：

```python
workflow.add_conditional_edges(
    "generate",                     # 来源节点
    should_continue,                # 条件函数
    path_map={"reflect": "reflect", "end": END}  # 目标映射
)
```

---

## 🪄 Step 6：编译与可视化

```python
graph = workflow.compile()
```

可使用以下方式导出可视化结构：

* Mermaid 图（适合导入 Excalidraw 或 Markdown）：

  ```python
  print(graph.get_graph().draw_mermaid())
  ```
* ASCII 图（命令行查看）：

  ```python
  graph.get_graph().print_ascii()
  ```

Mermaid 图例：

```
Start --> Generate
Generate --> Reflect
Reflect --> Generate
Generate --> End
```

---

## 🔁 Step 7：运行时执行逻辑

执行顺序如下：

1. **Generate Node** → 运行 `generate_chain`
2. 调用 `should_continue` 判断：

   * 若条件满足 → 跳至 **Reflect Node**
   * 若结束 → 跳至 **End Node**
3. **Reflect Node** → 运行 `reflect_chain`
4. 更新状态 → 再次回到 Generate Node
5. 重复循环直到 `should_continue` 返回 "end"

---

## ⚙️ Step 8：运行后的结果

运行时：

* 所有消息依次追加到 `state["messages"]`
* 模型每次调用都能看到完整历史上下文
* 可在 trace 中观察每次生成与反思的演进

---

## 💡 关键总结

| 模块                | 核心作用      | LangGraph 对应                             |
| ----------------- | --------- | ---------------------------------------- |
| **State**         | 保存所有上下文   | `TypedDict` + Reducer                    |
| **Node**          | 执行具体逻辑    | Chain 调用函数                               |
| **Edge**          | 定义执行路径    | `add_edge()` / `add_conditional_edges()` |
| **Reducer**       | 定义状态更新方式  | `add_messages`                           |
| **Flow Control**  | 控制何时继续/结束 | `should_continue()`                      |
| **Visualization** | 输出流程图     | `draw_mermaid()` / `print_ascii()`       |

---

