---
name: engineering-cybernetics-apply
description: "工程控制论应用：基于诊断结果设计稳定的控制闭环——从系统建模到控制策略、实施步骤到验证计划，七部分输出完整改造方案。"
version: 1.0.0
metadata:
  hermes:
    tags: [cybernetics, engineering, control-systems, feedback-design, 控制论, 工程改造, 系统优化]
    related_skills: [engineering-cybernetics-diagnose, planning-and-task-breakdown, incremental-implementation]
---

# 工程控制论应用 (Engineering Cybernetics Apply)

## 概述

基于钱学森《工程控制论》的工程改造方法论。不是头痛医头，而是从控制论角度为系统设计一个**更稳定的控制闭环**——重设控制器、优化反馈链路、隔离扰动、保障关键状态变量。配合 [[engineering-cybernetics-diagnose]] 使用：先诊断，再改造。

## 触发条件

- 诊断完成后需要出改造方案
- 用户说"怎么修""怎么改造这个系统"
- 设计新系统的控制架构
- AI Agent / 数据流水线的稳定性优化
- 用户提到"控制闭环""反馈优化""系统重构"等关键词

## 核心原则

> 改造不是修 bug，而是重新设计控制关系——让系统自己回到目标状态。

## 七部分改造方案输出

每次应用输出以下结构：

```
1. 控制目标 —— 系统要达到什么状态
2. 系统模型 —— 抽象的数学模型/逻辑模型
3. 推荐近似模型 —— 工程实用的简化模型
4. 控制策略 —— 反馈回路设计
5. 实施步骤 —— 具体改造方案
6. 验证计划 —— 如何确认改造有效
7. 风险和待确认事项 —— 不确定性清单
```

---

## 第 1 部分：控制目标

### 目标状态定义

用**可量化**的语言描述系统要达到的状态：

| 要素 | 描述 | 量化指标 |
|------|------|---------|
| 目标值 (Setpoint) | 期望的系统输出 | 准确率 > 95%、延迟 < 200ms |
| 允许误差 (Tolerance) | 可接受的偏差范围 | ±5% |
| 恢复时间 (Settling Time) | 扰动后回到目标的时间 | < 30s |
| 稳态误差 (Steady-State Error) | 长期运行的偏差 | < 1% |

### 目标分解

```python
class ControlObjective:
    def __init__(self):
        self.setpoint = {}      # 目标值
        self.tolerance = {}     # 允许误差
        self.settling_time = 0  # 恢复时间 (秒)
        self.steady_error = 0   # 稳态误差
    
    def define(self, name: str, target, tolerance, settling_time=30):
        self.setpoint[name] = target
        self.tolerance[name] = tolerance
        self.settling_time = settling_time
        return self
```

---

## 第 2 部分：系统模型

### 状态空间表示

将系统抽象为状态空间模型：

```
x[t+1] = f(x[t], u[t], w[t])   # 状态转移方程
y[t]   = g(x[t], v[t])         # 观测方程
```

其中：
- `x[t]` — 状态向量（系统内部状态）
- `u[t]` — 控制输入（可调节的参数/策略）
- `w[t]` — 过程扰动（外部不可控变化）
- `y[t]` — 观测输出（能测量到的结果）
- `v[t]` — 测量噪声（观测误差）

### AI 系统的状态空间映射

```python
class SystemModel:
    """工程控制论系统模型"""
    
    def __init__(self):
        self.states = []        # 状态变量
        self.controls = []      # 可控输入
        self.disturbances = []  # 扰动
        self.outputs = []       # 输出
        self.noise_sources = [] # 测量噪声
    
    def add_state(self, name, initial, domain):
        """添加状态变量"""
        self.states.append({'name': name, 'initial': initial, 'domain': domain})
    
    def add_control(self, name, default, range_):
        """添加可控输入"""
        self.controls.append({'name': name, 'default': default, 'range': range_})
    
    def add_disturbance(self, name, source, impact):
        """添加扰动源"""
        self.disturbances.append({'name': name, 'source': source, 'impact': impact})
    
    def add_output(self, name, formula):
        """添加输出"""
        self.outputs.append({'name': name, 'formula': formula})
```

---

## 第 3 部分：推荐近似模型

工程中不需要完美模型——选择**够用的近似**：

| 场景 | 推荐近似 | 理由 |
|------|---------|------|
| LLM 调用 | 概率延迟模型 | 响应时间是随机的 |
| 缓存层 | 命中率×新鲜度 | 两个参数足够 |
| 规则引擎 | 有限状态机 | 判定是离散的 |
| 消息队列 | 排队论 (M/M/1) | 泊松到达 + 指数服务 |
| 探活/健康检查 | 二值检测 + 误报率 | 有噪声的布尔信号 |

### 近似模型选择器

```python
def recommend_model(system_type: str, noise_level: str) -> str:
    models = {
        ('llm', 'high'): '概率延迟模型 + 输出分布拟合',
        ('llm', 'low'): '固定延迟 + 确定性输出',
        ('cache', 'any'): 'LRU 命中率模型 + TTL 新鲜度',
        ('rules', 'any'): '确定有限状态机 (DFA)',
        ('queue', 'any'): 'M/M/1/K 排队模型',
        ('health_check', 'high'): '伯努利试验 + 贝叶斯更新',
        ('health_check', 'low'): '简单阈值 + 连续失败计数',
    }
    return models.get((system_type, noise_level), '经验模型 + 在线校准')
```

---

## 第 4 部分：控制策略

### 控制策略选择树

```
系统能否承受振荡？
├── 是 → 高增益反馈 (快速收敛，允许超调)
└── 否 → 低增益反馈 (平滑收敛，慢但稳)

测量噪声大吗？
├── 是 → 滤波后再反馈 (移动平均/指数平滑/卡尔曼)
└── 否 → 直接反馈

扰动可预测吗？
├── 是 → 前馈控制 (在扰动影响前先补偿)
└── 否 → 反馈控制 + 自适应阈值

时滞严重吗？
├── 是 → Smith 预估器 (预测未来状态再决策)
└── 否 → 即时反馈
```

### P0 模式：网格搜索替代反馈回路

当系统的主优化器（如 EvoScientist/RL/贝叶斯优化）不可用时，用参数网格搜索 + 评分函数重建反馈：

```python
# 搜索空间：模块权重 × 增益
best_score = -inf
for wc in linspace(0.15, 0.5, 8):
    for wn in linspace(0.15, 0.5, 8):
        wf = 1.0 - wc - wn
        if wf < 0.1: continue
        for gain in [0.2, 0.3, 0.4, 0.5]:
            unified = wc * norm1 + wn * norm2 + wf * norm3
            correction = (unified - median(unified)) * gain * 2
            score = balance_score - extreme_ratio * 3 + min_weight * 2
            if score > best_score: save weights

# 关键: 中位数中心化, 避免固定中心 0.5 的偏斜陷阱
# 评分: 平衡正负 + 抑制极端 + 奖励均衡
```

适用条件：
- 模块数 2-5，搜索空间 < 1000 组合
- 每次评估 < 1s（纯 numpy，~3000 行数据）
- 有明确的评分函数

| 模式 | 描述 | 适用场景 |
|------|------|---------|
| **阈值控制** | 超阈值触发动作 | 告警、限流、熔断 |
| **PID 控制** | 比例-积分-微分 | 资源调度、自动扩缩容 |
| **自适应控制** | 参数在线调整 | 动态阈值、自适应超时 |
| **级联控制** | 外层控制内层 | 粗分类→细分类→判定 |
| **前馈控制** | 预测扰动提前补偿 | 流量预测→预扩容 |
| **冗余控制** | 多路表决 | LLM 多模型投票 |
| **鲁棒控制** | 对抗最坏情况 | 输入校验 + 异常降级 |
| **网格搜索调参** | 参数空间穷举评分 | 优化器不可用时的反馈替代方案 |
| **网格搜索替代反馈** | 参数网格搜索替代不可用的优化器 | EvoScientist ❌ → 网格搜索自动寻优权重 |

### 控制策略代码模板

```python
class FeedbackController:
    """通用反馈控制器"""
    
    def __init__(self, setpoint, kp=1.0, ki=0.0, kd=0.0):
        self.setpoint = setpoint  # 目标值
        self.kp = kp  # 比例增益
        self.ki = ki  # 积分增益
        self.kd = kd  # 微分增益
        self.prev_error = 0
        self.integral = 0
        self.filter_window = []
    
    def low_pass_filter(self, measurement, window_size=5):
        """低通滤波：平滑测量噪声"""
        self.filter_window.append(measurement)
        if len(self.filter_window) > window_size:
            self.filter_window.pop(0)
        return sum(self.filter_window) / len(self.filter_window)
    
    def compute(self, measurement, dt=1.0):
        """PID 控制量计算"""
        filtered = self.low_pass_filter(measurement)
        error = self.setpoint - filtered
        
        # 死区：小误差不触发
        if abs(error) < 0.01 * self.setpoint:
            return 0
        
        self.integral += error * dt
        derivative = (error - self.prev_error) / dt
        
        output = (self.kp * error 
                 + self.ki * self.integral 
                 + self.kd * derivative)
        
        self.prev_error = error
        return output


class CascadeController:
    """级联控制器：外层粗调 + 内层精调"""
    
    def __init__(self, outer_setpoint, inner_setpoint):
        self.outer = FeedbackController(outer_setpoint, kp=0.5, ki=0.1)
        self.inner = FeedbackController(inner_setpoint, kp=2.0, ki=0.3, kd=0.1)
    
    def step(self, outer_measurement, inner_measurement, dt=1.0):
        """级联控制：外层输出 → 内层目标"""
        inner_target = self.outer.compute(outer_measurement, dt)
        self.inner.setpoint += inner_target  # 外层微调内层目标
        control_action = self.inner.compute(inner_measurement, dt)
        return control_action
```

---

## 第 5 部分：实施步骤

### 改造实施模板

```
Phase 1: 隔离（不破坏现有功能）
├── 1.1 增加中间状态日志（不改代码逻辑）
├── 1.2 建立回归基线（连续跑 3 次，记录差异）
└── 1.3 细分错误码（区分环境异常 vs 业务异常）

Phase 2: 稳化（加保护层）
├── 2.1 探活状态码从二值改为多值
├── 2.2 给不稳定组件加缓存稳定层
├── 2.3 LLM 分类结果加低置信度标记
└── 2.4 实现优雅降级路径

Phase 3: 优化（改控制策略）
├── 3.1 硬阈值 → 自适应阈值
├── 3.2 单一判断 → 多路表决
├── 3.3 同步阻塞 → 异步 + 超时保护
└── 3.4 固定重试 → 指数退避 + 抖动

Phase 4: 闭环（建立自愈能力）
├── 4.1 自动检测反馈质量下降
├── 4.2 控制器参数在线调优
└── 4.3 异常模式识别 → 自动降级
```

---

## 第 6 部分：验证计划

### 控制论验证矩阵

| 测试类型 | 方法 | 通过标准 |
|---------|------|---------|
| **稳定性测试** | 注入脉冲扰动，观察恢复 | 30s 内回到目标 ±5% |
| **鲁棒性测试** | 参数漂移 ±20%，观察输出 | 输出偏差 < 10% |
| **一致性测试** | 同输入跑 3 次 | 结果 100% 一致 |
| **噪声抑制** | 注入 10% 测量噪声 | 判定不变 |
| **降级测试** | 逐个摘除外部依赖 | 优雅降级，不崩溃 |
| **恢复测试** | 长时间扰动后移除 | 自动恢复到目标状态 |

### 回归基线脚本

```python
def establish_baseline(func, inputs, runs=3):
    """建立回归基线：同一输入多次运行，检查一致性"""
    results = []
    for i in range(runs):
        result = func(inputs)
        results.append(result)
    
    # 检查一致性
    if len(set(str(r) for r in results)) > 1:
        inconsistencies = []
        for i in range(len(results)):
            for j in range(i+1, len(results)):
                if results[i] != results[j]:
                    inconsistencies.append((i, j, results[i], results[j]))
        return {
            'consistent': False,
            'inconsistencies': inconsistencies,
            'message': f'基线不一致：{len(inconsistencies)} 处差异'
        }
    return {'consistent': True, 'results': results}
```

---

## 第 7 部分：风险和待确认事项

### 风险清单

| 风险 | 概率 | 影响 | 缓解措施 |
|------|:--:|------|---------|
| 改造引入新不稳定 | 中 | 高 | Phase 1 只加日志不改逻辑 |
| 控制参数难调 | 中 | 中 | 先离线调参，再灰度上线 |
| 反馈延迟导致过调 | 高 | 中 | 用低通滤波 + Smith 预估 |
| 降级路径本身故障 | 低 | 高 | 降级比主路径更简单可靠 |
| 模型与实际偏差 | 高 | 中 | 在线校准 + 定期重评估 |

### Agent 生成时植入

当用工程控制论创建 AI Agent 时，在 System Prompt 中注入：

```
你是一个基于工程控制论的智能体。面对任何系统问题时，遵循以下原则：

1. 先识别：受控对象是什么？控制器在哪里？反馈链路通不通？
2. 再诊断：扰动来自哪里？测量是否可靠？后面规则有没有放大噪声？
3. 后改造：优先修反馈质量，再稳化不稳定组件，最后优化控制策略。
4. 验证：改造后跑三次回归基线，确认结果一致。
```

## 与其他技能的联动

- [[engineering-cybernetics-diagnose]]：先用 Diagnose 定位根因，再用 Apply 出方案
- [[planning-and-task-breakdown]]：将改造方案分解为可执行任务
- [[incremental-implementation]]：分阶段实施，每阶段验证
- [[doubt-driven-development]]：对每个控制决策保持怀疑

## 陷阱

1. **预检文件路径不能凭记忆写**：输入校验的文件路径必须从目标系统的 `load_data()` 函数或实际 `grep` 结果确认，不能写"印象中的路径"。本次 CityHDG 诊断中写 `data/中国城市社会经济统计数据1990-2023.xlsx` 但实际是 `cheng-shi-jing-ji-yu-ce1.csv`——导致首跑直接被预检拦截。
2. **控制参数修改后必须验证语法**：`python -c "import py_compile; py_compile.compile(path, doraise=True)"` 快速验语法，避免上线才发现 SyntaxError。
3. **检查点不要存在进程本地 `/tmp`**：应用级 checkpoint 应放在项目目录内，随代码仓库持久化。
4. **API fallback 不能设为静默终端态**：一旦回退到 mock，所有下游结果都被污染且无法追溯。改为重试+显式标记。
5. **固定中心化陷阱（归一化融合专用）**：`correction = (unified - 0.5) * gain` 假设 unified 中位数=0.5，但多模块归一化后的实际中位数可能只有 0.18——导致 88% 负修正。**必须用 `np.median(unified)` 动态中心化**。V12→V13 修复验证：正修正从 12% 恢复到 50%。
6. **CSV 保存时序陷阱**：计算新列→保存 CSV→再计算→忘记再保存 = 中间列丢失。在添加最终列后**显式再 save 一次**。
7. **网格搜索增益范围**：若用 `extreme_ratio * 3` 惩罚极端修正，低增益自然得分最高。将增益下限从 0.15 提高到 0.20，或用归一化惩罚（除以增益的平方根）消除增益-惩罚的耦合。
8. **幽灵模块检测**：多模块系统中某个模块输出与最终决策零相关（|r|<0.05）→ 它被计算了但从未参与融合。改法：归一化+网格搜索+中位数中心融合。详见 `references/three-module-unified-fusion.md`。
9. **微信文章抓取不可达**：`mp.weixin.qq.com/s/...` 链接需验证码（CAPTCHA），curl 直连、Jina AI (`r.jina.ai`)、GitHub 仓库搜索三种方式均可能失败。若用户分享微信文章链接且内容不可达，直接请用户贴关键段落——不要反复尝试不同抓取方式。

- 钱学森, 宋健.《工程控制论》. 科学出版社, 1980 (修订版)
- 微信公众号文章: "我把《工程控制论》做成了两个 AI Skill" (2026)
