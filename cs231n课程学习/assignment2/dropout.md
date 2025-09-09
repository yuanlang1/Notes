# Q3: Dropout

## Dropout 前向传播公式

假设输入为 \( x \in \mathbb{R}^{n} \)，丢弃概率为 \( p \)，保留概率为 \( 1 - p \)，则：

### 1. 生成掩码 (mask)

\[
\text{mask} = \frac{\mathbf{1}_{\{u < 1 - p\}}}{1 - p}, \quad u \sim \mathcal{U}(0, 1)
\]

其中 \( \mathbf{1}_{\{ \cdot \}} \) 表示取值为 1 的布尔掩码，满足条件为 1，不满足为 0。

---

### 2. 训练模式（Train Mode）

\[
\text{out} = x \odot \text{mask}
\]

其中 \( \odot \) 表示逐元素乘法。

---

### 3. 测试模式（Test Mode）

\[
\text{out} = x
\]

> 由于使用的是 **Inverted Dropout**，所以测试时不需要再对输出进行缩放。

---

##  Dropout 反向传播公式

### 训练模式（Train Mode）

\[
\frac{\partial L}{\partial x} = \frac{\partial L}{\partial \text{out}} \odot \text{mask}
\]

---

### 测试模式（Test Mode）

\[
\frac{\partial L}{\partial x} = \frac{\partial L}{\partial \text{out}}
\]

---

## 代码

```python
def dropout_forward(x, dropout_param):
    p, mode = dropout_param["p"], dropout_param["mode"]

    if "seed" in dropout_param:
        np.random.seed(dropout_param["seed"])

    mask = None
    out = None

    if mode == "train":
        # 生成x相同形状的随机数组，比较后为布尔数组
        mask = np.random.rand(*x.shape) < (1 - p)
        mask = mask / (1 - p)
        out = mask * x

    elif mode == "test":
        out = x

    cache = (dropout_param, mask)
    out = out.astype(x.dtype, copy=False)

    return out, cache


def dropout_backward(dout, cache):
    dropout_param, mask = cache
    mode = dropout_param["mode"]

    dx = None
    if mode == "train":
        dx = dout * mask

    elif mode == "test":
        dx = dout

    return dx

```