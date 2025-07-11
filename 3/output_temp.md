# 问题三 分类结果
创建并运行了一个Python脚本 `3/problem3_classification.py`，该脚本完成了以下任务：

1.  **加载数据**：读取了 `3/附件3_处理后_ALR.csv` 文件。
2.  **特征工程**：精确复制了问题二中模型的训练过程，包括：
    *   将ALR数据中的 `-inf` 值替换为0。
    *   根据“表面风化”情况创建了“风化”特征。
    *   创建了化学成分与风化情况的交互特征。
3.  **加载模型**：加载了 `2/logistic_regression/` 和 `2/decision_tree/` 目录中预先训练好的逻辑回归模型、决策树模型以及它们各自对应的数据缩放器（scaler）。
4.  **进行预测**：
    *   使用逻辑回归模型预测了每个文物的类别，并得出了其判定为“高钾”的概率。
    *   使用决策树模型对相同的文物进行了分类，以进行交叉验证。
5.  **保存结果**：所有预测结果都已汇总并保存到新文件 `3/prediction_results.csv` 中。

以下是最终的分类结果：

```csv
文物编号,逻辑回归预测,逻辑回归预测为高钾的概率,决策树预测
A1,高钾,0.9396739866335334,高钾
A2,铅钡,0.2863112041099779,高钾
A3,铅钡,0.1199919487797667,铅钡
A4,铅钡,0.24620914588062723,铅钡
A5,铅钡,0.020770428199432135,铅钡
A6,高钾,0.9214413423228262,高钾
A7,高钾,0.9785582412503621,高钾
A8,铅钡,0.35545211142366434,铅钡
```

从结果中可以看到，两种模型的预测结果在大部分样本上是一致的。对于文物 `A2`，逻辑回归模型预测为“铅钡”（概率约为28.6%），而决策树模型预测为“高钾”，这可能是一个需要进一步进行敏感性分析的样本点。

如果您需要进行后续的敏感性分析或者有其他问题，请随时提出。

## 敏感性分析

以下是结果的总结和解读：

### 1. 自助法（Bootstrap）敏感性分析

这份报告衡量了当训练数据发生微小变化时，预测结果的稳定性。

```
--- 自助法敏感性分析报告 ---
  文物编号 原始预测  高钾概率的标准差  与原始预测一致率
0   A1     高钾      0.176026       0.946
1   A2     铅钡      0.174831       0.868
2   A3     铅钡      0.093239       0.988
3   A4     铅钡      0.109561       0.986
4   A5     铅钡      0.102531       0.986
5   A6     高钾      0.047573       1.000
6   A7     高钾      0.302624       0.884
7   A8     铅钡      0.165575       0.910
```

**解读**:
*   **高钾概率的标准差**：这个值越小，说明模型的预测概率越稳定。文物 `A6` 的标准差最小（0.048），说明模型对它的判断非常一致。文物 `A7` 的标准差最大（0.303），说明不同模型对它的判断摇摆不定，敏感性较高。
*   **与原始预测一致率**：这个比例越高，说明预测结果越可靠。文物 `A6` 的预测结果在所有500次测试中都保持不变（100%一致），非常稳健。相比之下，文物 `A2` 的一致率最低（86.8%），说明有超过13%的情况下，不同的训练数据会导致其预测结果从“铅钡”翻转为“高钾”，这与我们在第一步的发现（决策树预测它为高钾）相符，进一步证明了其分类结果的敏感性。

### 2. 数据扰动法敏感性分析

这份报告衡量了每个样本的化学成分数据在多大程度上被噪声干扰后，其分类结果依然保持不变。

```
--- 数据扰动法敏感性分析报告 ---
  文物编号 原始预测  能容忍的最大噪声水平(ε)
0   A1     高钾           0.50
1   A2     铅钡           0.50
2   A3     铅钡           0.50
3   A4     铅钡           0.50
4   A5     铅钡           0.46
5   A6     高钾           0.50
6   A7     高钾           0.50
7   A8     铅钡           0.43
```

**解读**:
*   **能容忍的最大噪声水平(ε)**：这个值越大，说明分类结果对数据本身的微小变动或测量误差越不敏感，结果越稳健。
*   大部分样本（如 `A1`, `A2`, `A3`, `A4`, `A6`, `A7`）都非常稳健，即使在特征值上增加了高达50%的噪声，其分类结果也没有改变。
*   文物 `A5` 和 `A8` 的分类结果相对敏感一些，分别在噪声水平达到46%和43%时发生了改变。

### 结论

综合两种分析：
*   **最稳健的样本**：文物 `A6` 在两种分析中都表现出极高的稳定性，其分类为“高钾”的结果非常可靠。
*   **最敏感的样本**：文物 `A2` 和 `A7` 值得特别关注。`A2` 在自助法分析中表现出较高的不一致性，`A7` 的预测概率波动最大。这表明它们的化学成分特征可能处于分类边界附近，微小的变化就可能影响其分类结果。

所有分析报告都已保存在 `3/` 目录下，分别为 `sensitivity_bootstrap_report.csv` 和 `sensitivity_perturbation_report.csv`。任务已完成。