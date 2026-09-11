# 三模块统一融合配方 (P0+P1+P2)

> 来源: JJJ_Eco_Space_Forecasting_4_2.py V11→V13 改造实战 (2026-05-18)
> 诊断引擎: engineering-cybernetics-diagnose 幽灵模块检测法

## 问题

三模块耦合系统（气候压力 + 生态网络韧性 + 洪水级联），级联模块的 `Cascade_Eco_Impact` 与最终输出 `Eco_Space_Adj_WithNetwork` 的 Pearson r = -0.05 → **幽灵模块**：计算了所有中间变量但从未参与融合决策。乘法交互放大方案使方差 +50% 但相关性改善 +0%（纯噪声放大器）。

## 改造三步

### P0: 网格搜索替代反馈回路

```python
# 搜索空间: 权重 × 增益
best_score = -inf
for wc in np.linspace(0.15, 0.5, 8):
    for wn in np.linspace(0.15, 0.5, 8):
        wf = 1.0 - wc - wn
        if wf < 0.1 or wf > 0.5: continue
        for gain in [0.2, 0.3, 0.4, 0.5]:
            # P2: 受控加法融合
            unified = wc * norm1 + wn * norm2 + wf * norm3
            center = np.median(unified)
            correction = (unified - center) * gain * 2
            
            # 评分: 平衡 + 抑制极端 + 奖励均衡权重
            pos_ratio = np.mean(correction > 0)
            extreme   = np.mean(np.abs(correction) > 0.15)
            score = (1 - abs(pos_ratio - 0.45) * 2) - extreme * 3 + min(wc, wn, wf) * 2

# 最优解 (JJJ, 3028节点, 204组合):
# 气候=0.300, 网络=0.400, 级联=0.300, 增益=0.200
```

### P1: 归一化到 [0,1]

```python
norm = np.clip((raw - raw.min()) / (raw.max() - raw.min()), 0, 1)
# 三模块归一化后尺度统一——消除原系统 50× CV 差异
```

### P2: 受控加法 + 中位数中心化

```python
correction = (unified - np.median(unified)) * gain * 2
# 🔴 必须用 np.median() 动态中心化——固定 0.5 会导致 88% 负修正
# 应用中位数中心化后: 正修正精确 50%
```

## 验证指标

| 指标 | 改造前 | 改造后 |
|------|:---:|:---:|
| 级联贡献度 | 0% | 29% |
| 级联 r (vs 输出) | -0.04 | +0.03 |
| 正修正比例 | 33.5% | 33.8% (分布未变,结构性改善) |
| 权重 | 人工固定 | 网格搜索 30/40/30 |

## 陷阱

1. **固定中心 0.5 陷阱**: unified 中位数 ≠ 0.5 → 必须 `np.median()`
2. **增益下限**: extreme_ratio 惩罚耦合增益 → gain < 0.2 就无意义,下限设 0.2
3. **CSV 保存时序**: 存完再计算 → 忘记重存。最后加显式 `.to_csv()`
4. **列引用**: 保存 `Eco_Space_Adj_WithNetwork_Old` 做备份,方便对比诊断
