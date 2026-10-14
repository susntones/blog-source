---
title: mini-SWE-agent 学习笔记：100 行代码的编码 Agent
tags:
  - AI
  - Agent
  - 源码解读
categories:
  - AI学习
date: 2026-10-14 20:00:00
---

## 简介 ##

[mini-SWE-agent](https://github.com/SWE-agent/mini-swe-agent) 是 Princeton / Stanford 的 SWE-bench、SWE-agent 团队做的“极简版”编码 Agent。他们在 README 里抛出的问题很有意思：

> What if our agent was 100x simpler, and still worked nearly as well?

结论是：**一个只会执行 bash 的 Agent，核心类约 100 行 Python，在 SWE-bench Verified 上能拿到 74% 以上的分数。** 它现在被很多团队拿来当评测基线、做 RL 训练的 rollout 框架，也可以当日常命令行助手用。

本文基于 **v2.4.6**（commit `04d809c`，2026-09）源码整理，分三部分：

1. 核心能力：它能做什么、怎么用
2. 原理解析：为什么“只有 bash”也够用
3. 源码解读：Agent / Model / Environment 三层逐个拆

<!-- more -->

---

## 一、核心能力 ##

### 1.1 一句话定位 ###

一个 **ReAct 风格的单循环 Agent**：把任务交给 LLM，LLM 每一轮输出“思考 + bash 命令”，框架执行命令后把输出喂回去，直到 LLM 发出约定好的“提交”命令为止。

### 1.2 能力清单 ###

| 能力 | 说明 |
|------|------|
| **唯一工具：bash** | 没有 `edit_file`、`search` 等专用工具，读写文件、搜索、跑测试全靠 shell 命令（`cat`/`sed`/`grep`/heredoc） |
| **多执行环境** | 本地 `local`、`docker`/`podman`、`singularity`/apptainer、`bubblewrap`、`contree`、SWE-ReX（Modal 等远程沙箱） |
| **多模型** | 默认走 litellm，兼容几乎所有厂商；另有 openrouter、portkey、requesty，以及 `/response` API 版本 |
| **两种动作格式** | v2 默认用 **tool calling**（一个 `bash` 工具）；也保留 v1 的 **纯文本 + 正则提取** 模式（`*_textbased` 模型类） |
| **交互模式** | `mini` 命令行有 `confirm`（逐条确认）、`yolo`（直接执行）、`human`（人来敲命令）三种模式，运行中可 `Ctrl+C` 打断补充指示 |
| **成本/步数/时间限制** | `step_limit`、`cost_limit`、`wall_time_limit_seconds`，以及进程级的全局限额 |
| **完整轨迹** | 每一步都把完整 messages 落盘为 `*.traj.json`，可用 `mini-extra inspector` 浏览，天然适合做 SFT/RL 数据 |
| **批量评测** | 内置 SWE-bench（full/verified/lite/multimodal/multilingual…）、ProgramBench 的多线程批量运行脚本 |

### 1.3 快速上手 ###

```bash
pip install mini-swe-agent

# 首次运行会引导设置默认模型和 API Key
mini

# 指定模型和任务，跳过确认
mini -m anthropic/claude-sonnet-4-5-20250929 -t "给这个项目加上 pytest 配置" -y

# 用 Python 直接调用（官方 hello_world.py 的精简版）
```

```python
from minisweagent.agents.default import DefaultAgent
from minisweagent.environments.local import LocalEnvironment
from minisweagent.models.litellm_model import LitellmModel

agent = DefaultAgent(
    LitellmModel(model_name="anthropic/claude-sonnet-4-5-20250929"),
    LocalEnvironment(),
    system_template="You are a helpful assistant that can interact with a computer.",
    instance_template="Please solve this issue: {{task}} ...",
)
agent.run("Fix the failing test in tests/test_utils.py")
```

跑 SWE-bench：

```bash
mini-extra swebench --subset verified --split test -m <model> -w 8 -o ./runs/verified
```

---

## 二、原理解析 ##

### 2.1 整体架构：三个协议 + 一个循环 ###

整个框架只有三种核心组件，在 `minisweagent/__init__.py` 里用 `typing.Protocol` 定义（鸭子类型，不需要继承）：

```
              ┌────────────────────────────────────────┐
              │                Agent                   │
              │  messages: list[dict]   ← 唯一的状态    │
              │                                        │
   task ───▶  │  run(): while not exit: step()          │ ──▶ trajectory.json
              │  step() = execute_actions(query())     │
              └───────┬─────────────────────┬──────────┘
                      │ query(messages)     │ execute(action)
                      ▼                     ▼
              ┌──────────────┐      ┌────────────────┐
              │    Model     │      │  Environment   │
              │ LLM 调用      │      │ 执行 bash       │
              │ 解析 action   │      │ 返回 output     │
              │ 格式化观测     │      │ 检测“提交”信号   │
              └──────────────┘      └────────────────┘
```

- **Model**：负责“和 LLM 说话”——发请求、算成本、把响应解析成 action 列表、把命令输出渲染成 observation 消息。
- **Environment**：负责“执行命令”——`execute({"command": ...})` 返回 `{output, returncode, exception_info}`。
- **Agent**：只做编排——维护消息列表、检查限额、调用 Model 和 Environment、处理异常。

### 2.2 为什么只给 bash 就够了 ###

这是 mini-SWE-agent 最核心的设计判断。原版 SWE-agent 花了大量精力设计 ACI（Agent-Computer Interface）：专门的文件查看器、带 lint 的编辑命令、搜索工具等。2024 年这些工具对弱模型帮助很大，但随着模型变强：

1. **现代 LLM 本身就精通 shell**。`grep -rn`、`sed -i`、`cat <<'EOF'`、`nl -ba file | sed -n '10,20p'` 这些在训练数据里海量存在，模型用起来比自定义工具更自然。
2. **专用工具 = 额外的 prompt 负担和出错面**。每加一个工具就要写说明、处理参数错误、维护实现。
3. **无状态执行带来可移植性**。每条命令都是独立的 `subprocess.run`（或 `docker exec`），不维护常驻 shell 会话。这意味着：
   - 切换沙箱只需替换一行“怎么执行命令”的实现
   - 不会出现 shell 卡死在交互程序、环境变量被污染等状态问题
   - 代价是 `cd`、`export` 不会持久，prompt 里明确告诉模型：用 `cd /path && ...` 前缀

   这也是 prompt 里“Directory or environment variable changes are not persistent. Every action is executed in a new subshell.” 这句话的来由。

### 2.3 线性历史（Linear History） ###

**Agent 发给 LLM 的消息 = Agent 保存的轨迹。** 没有摘要压缩、没有历史裁剪、没有隐藏的 scratchpad。好处：

- 调试简单：看 `traj.json` 就知道模型每一步看到了什么
- 训练友好：轨迹直接就是 SFT/RL 的样本，不存在“训练分布和推理分布不一致”
- 代价：长任务可能撑爆上下文。框架的处理方式是**截断单条观测**（输出超过 10000 字符只保留头尾各 5000），而不是截断历史

### 2.4 控制流：用异常驱动 ###

循环何时结束、出错怎么办，全部用异常表达，这是代码能写这么短的关键：

```
InterruptAgentFlow(*messages)        # 基类：携带要追加到历史的消息
├── Submitted                        # 模型发出提交命令 → role=exit
├── LimitsExceeded                   # 步数/成本超限   → role=exit
│   └── TimeExceeded                 # 墙钟超时       → role=exit
├── UserInterruption                 # 用户打断/拒绝  → role=user（继续循环）
└── FormatError                      # 模型输出格式错 → role=user（继续循环，带错误提示）
```

统一约定：**异常里带着消息，Agent 捕获后把消息追加进历史；只要最后一条消息 `role == "exit"`，循环结束。** 这样“格式错误让模型重试”“用户插话”“任务提交”在主循环里是同一种处理路径。

### 2.5 完成协议：魔法字符串 ###

模型怎么告诉框架“我做完了”？没有 `submit` 工具，而是约定一行输出：

```bash
echo COMPLETE_TASK_AND_SUBMIT_FINAL_OUTPUT && cat patch.txt
```

Environment 执行完命令后检查：**输出第一行是这个字符串且 returncode 为 0**，就抛 `Submitted`，第一行之后的内容作为 submission（SWE-bench 场景下就是 git patch）。好处是“提交”也走同一条 bash 通道，模型可以先生成 patch、检查、再提交。

### 2.6 一次完整交互的消息序列 ###

```
[system]    You are a helpful assistant ...
[user]      Please solve this issue: {{task}}  + 工作流建议 + 命令示例
[assistant] 我先看看目录结构 ...   tool_call: bash {"command": "ls -la"}
[tool]      {"returncode": 0, "output": "..."}
[assistant] ...                    tool_call: bash {"command": "grep -rn foo src/"}
[tool]      ...
   ...（如果模型没调工具 → [user] Tool call error: ... 提示重试）
[assistant] tool_call: bash {"command": "echo COMPLETE_TASK_AND_SUBMIT_FINAL_OUTPUT"}
[exit]      exit_status=Submitted, submission=...
```

注意最后的 `exit` 角色是框架内部的，不会发给 LLM（发送前会过滤 `extra` 字段，`exit` 消息只出现在轨迹里）。

---

## 三、源码解读 ##

### 3.1 目录结构 ###

```
src/minisweagent/
├── __init__.py            # 版本号、全局配置目录、三个 Protocol
├── exceptions.py          # 控制流异常（20 行）
├── agents/
│   ├── default.py         # ★ DefaultAgent（190 行，核心）
│   └── interactive.py     # InteractiveAgent：人在回路
├── models/
│   ├── __init__.py        # get_model 工厂、全局成本统计
│   ├── litellm_model.py   # ★ 默认模型实现（tool calling）
│   ├── litellm_textbased_model.py  # 文本 + 正则模式
│   └── utils/             # action 解析、cache_control、重试...
├── environments/
│   ├── local.py           # ★ 本地 subprocess
│   ├── docker.py          # docker run -d + docker exec
│   └── extra/             # bubblewrap / swerex / contree ...
├── config/
│   ├── mini.yaml          # mini 命令的 prompt 和参数
│   └── benchmarks/swebench.yaml
└── run/
    ├── hello_world.py     # 最小可运行示例
    ├── mini.py            # `mini` CLI
    └── benchmarks/swebench.py  # 批量评测
```

整个包大约 5300 行，其中真正的“Agent 逻辑”集中在三个带 ★ 的文件。

### 3.2 DefaultAgent：主循环 ###

配置用 pydantic 定义，所有字段都能从 yaml 注入：

```python
class AgentConfig(BaseModel):
    system_template: str              # 第一条 system 消息（Jinja2 模板）
    instance_template: str            # 第二条 user 消息，描述任务
    step_limit: int = 0               # 0 表示不限
    cost_limit: float = 3.0           # 美元
    wall_time_limit_seconds: int = 0
    max_consecutive_format_errors: int = 3
    output_path: Path | None = None
```

`run()` 是整个框架的心脏：

```python
def run(self, task: str = "", **kwargs) -> dict:
    self.extra_template_vars |= {"task": task, **kwargs}
    self.messages = []
    self.add_messages(
        self.model.format_message(role="system", content=self._render_template(self.config.system_template)),
        self.model.format_message(role="user", content=self._render_template(self.config.instance_template)),
    )
    while True:
        try:
            self.step()
            self.n_consecutive_format_errors = 0
        except FormatError as e:
            # 格式错误也已经计费了，要把成本记上
            self.cost += e.messages[0].get("extra", {}).get("cost", 0.0)
            self.n_consecutive_format_errors += 1
            if 0 < self.config.max_consecutive_format_errors <= self.n_consecutive_format_errors:
                self.add_messages(*e.messages, {"role": "exit", "content": "RepeatedFormatError", ...})
            else:
                self.add_messages(*e.messages)
        except InterruptAgentFlow as e:
            self.add_messages(*e.messages)      # Submitted / LimitsExceeded / UserInterruption
        except Exception as e:
            self.handle_uncaught_exception(e)   # 记一条 exit 消息再抛出
            raise
        finally:
            self.save(self.config.output_path)  # 每一步都落盘
        if self.messages[-1].get("role") == "exit":
            break
    return self.messages[-1].get("extra", {})
```

几个值得注意的细节：

- **`finally: self.save(...)`**：每一步都写轨迹文件，进程崩了也不丢数据。
- **连续格式错误保护**：模型连续 3 次不调工具就退出，防止无限空转烧钱。
- **未知异常先记录再 raise**：轨迹里会留下 traceback，方便排查。

`step()` 只有一行，`query()` 和 `execute_actions()` 也很短：

```python
def step(self) -> list[dict]:
    return self.execute_actions(self.query())

def query(self) -> dict:
    if 0 < self.config.step_limit <= self.n_calls or 0 < self.config.cost_limit <= self.cost:
        raise LimitsExceeded({"role": "exit", "content": "LimitsExceeded", ...})
    if 0 < self.config.wall_time_limit_seconds <= int(time.time() - self._start_time):
        raise TimeExceeded({...})
    self.n_calls += 1
    message = self.model.query(self.messages)
    self.cost += message.get("extra", {}).get("cost", 0.0)
    self.add_messages(message)
    return message

def execute_actions(self, message: dict) -> list[dict]:
    outputs = [self.env.execute(action) for action in message.get("extra", {}).get("actions", [])]
    return self.add_messages(*self.model.format_observation_messages(message, outputs, self.get_template_vars()))
```

**限额检查放在调用 LLM 之前**——超限时连这次请求都不发。`0 < limit <= value` 的写法让 `0` 天然表示“不限制”。

模板变量来自四方合并（后者覆盖前者）：

```python
def get_template_vars(self, **kwargs) -> dict:
    return recursive_merge(
        self.config.model_dump(),          # agent 配置
        self.env.get_template_vars(),      # 环境：platform.uname()、环境变量等
        self.model.get_template_vars(),    # 模型配置
        {"n_model_calls": ..., "model_cost": ..., "elapsed_seconds": ...},
        self.extra_template_vars,          # task 等
        kwargs,
    )
```

所以 prompt 里能写 `{{system}} {{release}} {{machine}}`，甚至 `{% if system == "Darwin" %}` 来提示 macOS 上 `sed -i ''` 的差异。模板用 `StrictUndefined`，变量拼错会直接报错而不是静默渲染成空串。

### 3.3 Model：请求、解析、观测格式化 ###

`LitellmModel.query()` 的流程：

```python
def query(self, messages, **kwargs) -> dict:
    for attempt in retry(logger=logger, abort_exceptions=self.abort_exceptions):
        with attempt:
            response = self._query(self._prepare_messages_for_api(messages), **kwargs)
    cost_output = self._calculate_cost(response)
    GLOBAL_MODEL_STATS.add(cost_output["cost"])
    try:
        actions = self._parse_actions(response)
    except FormatError as e:
        e.messages[0]["extra"].update(cost_output)   # 解析失败也要把成本和原始响应带出去
        e.messages[0]["extra"]["response"] = response.model_dump(mode="json")
        raise
    message = response.choices[0].message.model_dump()
    message["extra"] = {"actions": actions, "response": response.model_dump(), **cost_output, "timestamp": time.time()}
    return message
```

**`extra` 字段约定**是贯穿全框架的小技巧：每条消息都可以挂一个 `extra` 字典存放元数据（actions、cost、原始响应、exit_status…），发 API 前统一剥掉：

```python
def _prepare_messages_for_api(self, messages):
    prepared = [{k: v for k, v in msg.items() if k != "extra"} for msg in messages]
    prepared = _reorder_anthropic_thinking_blocks(prepared)
    return set_cache_control(prepared, mode=self.config.set_cache_control)
```

这样消息列表既是 API 输入，又是带完整元数据的轨迹，一份数据两用。

**重试**用 tenacity，指数退避 4s~60s，默认最多 10 次；认证失败、上下文超长、参数不支持这类“重试也没用”的异常放在 `abort_exceptions` 里直接失败。

**Action 解析**（tool calling 模式）：只注册一个工具

```python
BASH_TOOL = {
    "type": "function",
    "function": {
        "name": "bash",
        "description": "Execute a bash command",
        "parameters": {"type": "object", "properties": {"command": {"type": "string", ...}}, "required": ["command"]},
    },
}
```

`parse_toolcall_actions` 检查三种错误：没有工具调用、参数不是合法 JSON、工具名不是 `bash`。任何一种都抛 `FormatError`，错误消息用 `format_error_template` 渲染。`mini.yaml` 里的模板还会区分 `finish_reason == "length"`——是模型输出被截断了，就提示“说简短点”，而不是笼统地说格式错误。

一轮可以有**多个** tool call，会依次执行，每个输出对应一条 `role: tool` 消息（用 `tool_call_id` 关联）；如果中途某条命令触发了提交或被用户拒绝，没执行的会被补上 `"action was not executed"`，保证 tool call 和 tool result 一一对应（否则 API 会报错）。

**文本模式**（`LitellmTextbasedModel`）只是覆写了两个方法：不传 `tools`，用正则 ```` ```mswea_bash_command\n(.*?)\n``` ```` 从回复文本里抽命令，并要求**恰好一个**。这是 v1 和原版 SWE-agent 的做法，适合不支持 function calling 的模型。

**观测格式化**由 `observation_template` 控制，`mini.yaml` 中的版本：

```jinja
{%- if output.output | length < 10000 -%}
{ "returncode": {{ output.returncode }}, "output": {{ output.output | tojson }} }
{%- else -%}
{ "returncode": ..., "output_head": {{ output.output[:5000] | tojson }},
  "output_tail": {{ output.output[-5000:] | tojson }},
  "elided_chars": ..., "warning": "Output too long." }
{%- endif -%}
```

长输出“掐头去尾留中间省略”，这是线性历史下控制上下文长度的主要手段。

**Prompt Cache**：`get_model` 发现模型名含 `claude`/`anthropic` 等关键字时，自动开启 `set_cache_control="default_end"`——每次请求只在**最后一条消息**上打 `cache_control: ephemeral` 标记。因为历史是只追加的，上一轮的前缀天然命中缓存，长轨迹能省下大量成本。

### 3.4 Environment：执行与提交检测 ###

`LocalEnvironment` 的核心是一个比 `subprocess.run` 多做一件事的 `_run`：

```python
def _run(command, cwd, env, timeout):
    process = subprocess.Popen(
        command, shell=True, text=True, cwd=cwd, env=env,
        stdout=subprocess.PIPE, stderr=subprocess.STDOUT,   # stderr 合并进 stdout
        start_new_session=os.name == "posix",               # 新进程组
    )
    try:
        stdout, _ = process.communicate(timeout=timeout)
    except subprocess.TimeoutExpired:
        os.killpg(process.pid, signal.SIGKILL)              # 超时杀整个进程组
        stdout, _ = process.communicate()
        raise subprocess.TimeoutExpired(command, timeout, output=stdout)
    return subprocess.CompletedProcess(command, process.returncode, stdout=stdout)
```

**`start_new_session` + `killpg`** 是关键：模型执行 `python server.py &` 或 `npm test` 之类会派生子进程的命令时，超时后能把整棵进程树杀干净，不留孤儿进程。超时不会让 Agent 崩溃，而是把已有输出和异常信息作为观测返回给模型，让它自己调整策略。

`execute()` 最后调用 `_check_finished`：

```python
def _check_finished(self, output: dict):
    lines = output.get("output", "").lstrip().splitlines(keepends=True)
    if lines and lines[0].strip() == "COMPLETE_TASK_AND_SUBMIT_FINAL_OUTPUT" and output["returncode"] == 0:
        submission = "".join(lines[1:])
        raise Submitted({"role": "exit", "content": submission,
                         "extra": {"exit_status": "Submitted", "submission": submission}})
```

`returncode == 0` 的条件很重要：`echo COMPLETE... && cat patch.txt` 中如果 `patch.txt` 不存在，整条命令失败，就不会误提交。

`DockerEnvironment` 的思路完全一致，只是执行方式换了：

```
初始化:  docker run -d --name minisweagent-xxxx -w <cwd> --rm <image> sleep 2h
每条命令: docker exec -w <cwd> -e K=V <container_id> bash -lc "<command>"
清理:    docker stop / rm -f（后台异步执行，__del__ 里也会调用）
```

容器用 `sleep 2h` 保活，命令之间文件系统状态保留，但 shell 状态不保留——和本地环境语义一致。换成 singularity、bubblewrap、Modal 也只是换这几条命令。

### 3.5 InteractiveAgent：人在回路 ###

`mini` 命令默认用的是 `InteractiveAgent`，它通过覆写 `DefaultAgent` 的几个方法实现交互，没有改主循环：

| 覆写方法 | 增加的行为 |
|---------|-----------|
| `add_messages` | 每条消息打印到终端（带步数和成本） |
| `query` | `human` 模式下读用户输入当命令；超限时询问新的限额（非 TTY 环境直接退出） |
| `step` | 捕获 `KeyboardInterrupt`，让用户输入补充说明，转成 `UserInterruption` |
| `execute_actions` | `confirm` 模式下先让用户确认，输入任意文字即拒绝并作为反馈发给模型；模型要提交时可追加新任务 |

用户的拒绝/插话都通过抛 `UserInterruption` 变成一条 `role: user` 消息，主循环不需要任何改动——异常驱动控制流的扩展性在这里体现得很明显。`whitelist_actions` 支持正则白名单，匹配的命令免确认。

### 3.6 配置系统 ###

`mini` CLI 的配置来自多个 spec 的递归合并：

```bash
mini -c mini.yaml -c model.model_kwargs.temperature=0 -c agent.step_limit=50
```

- `get_config_from_spec`：带 `=` 的解析为嵌套字典（值尝试按 JSON 解析），否则按 yaml 路径查找（当前目录 → `$MSWEA_CONFIG_DIR` → 内置 config 目录）
- `recursive_merge`：后者覆盖前者，嵌套 dict 递归合并；用哨兵对象 `UNSET` 表示“命令行没传”，从而不会用 `None` 覆盖 yaml 里的值

最终 yaml 分成 `agent` / `model` / `environment` / `run` 四段，分别作为 kwargs 传给对应的类。类本身则用 `xxx_class` 字段 + 简写映射表（`"docker"` → `minisweagent.environments.docker.DockerEnvironment`）动态加载，也接受完整 import 路径，方便接入自定义实现。

### 3.7 SWE-bench 批量运行 ###

`run/benchmarks/swebench.py` 展示了框架在评测场景的用法：

1. 从 HuggingFace 加载数据集，按 `--filter` 正则和 `--slice` 筛选实例
2. `ThreadPoolExecutor(max_workers=workers)` 并发处理
3. 每个实例：根据 `instance_id` 推出镜像名 `swebench/sweb.eval.x86_64.<id>`，启动 Docker 环境，创建 `ProgressTrackingAgent`（只覆写了 `step` 用于更新进度条）
4. `agent.run(problem_statement)`，submission 即 git patch
5. 结果加锁写入 `preds.json`，轨迹写到 `<instance_id>/<instance_id>.traj.json`

`swebench.yaml` 的 prompt 要求模型先 `git diff -- <files> > patch.txt` 检查补丁，再单独执行 `echo COMPLETE_TASK_AND_SUBMIT_FINAL_OUTPUT && cat patch.txt` 提交；`step_limit: 250`、`cost_limit: 3`、命令超时 60 秒。

---

## 四、设计取舍与启发 ##

| 设计 | 收益 | 代价 |
|------|------|------|
| 只有 bash | 模型熟悉、无需维护工具、易移植 | 编辑大文件依赖 sed/heredoc，对弱模型不友好 |
| 无状态执行 | 沙箱可替换、无会话污染、易并行 | `cd`/`export` 不持久，需要 prompt 约束 |
| 线性历史 | 调试直观、轨迹即训练数据 | 超长任务受上下文窗口限制 |
| 异常驱动控制流 | 主循环极短，扩展只需抛异常 | 控制流不够显式，初读需理清异常层级 |
| 魔法字符串提交 | 复用 bash 通道，提交前可自检 | 依赖 prompt 约定，模型可能用错 |
| `extra` 元数据字段 | 一份消息列表兼作 API 输入和完整日志 | 每个 Model 实现都要记得剥离 |

几点个人体会：

1. **复杂度应该放在模型里，而不是脚手架里。** 模型能力提升后，很多精心设计的工具反而变成负担。先做最简单的版本作为基线，再证明每个额外组件的价值。
2. **Protocol + 组合优于深继承。** Agent、Model、Environment 三者正交，任意组合；想加功能就继承覆写一两个方法（`InteractiveAgent`、`ProgressTrackingAgent` 都是例子）。
3. **可观测性从第一天就要有。** 每一步落盘、成本精确统计、格式错误也计费，这些“无聊”的细节决定了它能否用于大规模评测和训练。
4. **自己写 Agent 时可以直接借鉴的点**：进程组级别的超时清理、长输出头尾截断、`0 < limit <= value` 的“0 即不限”写法、`StrictUndefined` 模板、`UNSET` 哨兵做配置合并、只在最后一条消息打 cache 标记。

## 参考 ##

- 仓库：https://github.com/SWE-agent/mini-swe-agent
- 文档：https://mini-swe-agent.com/latest/
- 控制流图解：https://mini-swe-agent.com/latest/advanced/control_flow/
- 从零构建极简 Agent 教程：https://minimal-agent.com/
- SWE-agent 论文：*SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering*（NeurIPS 2024）
