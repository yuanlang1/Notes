# Q2: Batch Normalization

##  Batch Normalization 前向传播与反向传播公式

###  前向传播（Forward Pass）

设输入为 $x \in \mathbb{R}^{N \times D}$，其中：

- $N$ 表示 batch 的大小；
- $D$ 表示每个样本的维度；
- $\gamma, \beta \in \mathbb{R}^D$ 是可学习参数；
- $\epsilon$ 是用于数值稳定的一个很小的常数。

公式如下：

1. **计算均值（mean）**：

$$
\mu = \frac{1}{N} \sum_{i=1}^{N} x_i
$$

2. **计算方差（variance）**：

$$
\sigma^2 = \frac{1}{N} \sum_{i=1}^{N} (x_i - \mu)^2
$$

3. **归一化（normalize）**：

$$
\hat{x}_i = \frac{x_i - \mu}{\sqrt{\sigma^2 + \epsilon}}
$$

4. **缩放与偏移（scale & shift）**：

$$
y_i = \gamma \hat{x}_i + \beta
$$

---

###  反向传播（Backward Pass）

假设损失为 $L$，需要求 $\frac{\partial L}{\partial x}$、$\frac{\partial L}{\partial \gamma}$ 和 $\frac{\partial L}{\partial \beta}$：

1. **对 $\beta$ 的梯度**：

$$
\frac{\partial L}{\partial \beta} = \sum_{i=1}^{N} \frac{\partial L}{\partial y_i}
$$

2. **对 $\gamma$ 的梯度**：

$$
\frac{\partial L}{\partial \gamma} = \sum_{i=1}^{N} \frac{\partial L}{\partial y_i} \cdot \hat{x}_i
$$

3. **对 $\hat{x}_i$ 的中间梯度**：

$$
\frac{\partial L}{\partial \hat{x}_i} = \frac{\partial L}{\partial y_i} \cdot \gamma
$$

4. **对方差的梯度**：

$$
\frac{\partial L}{\partial \sigma^2} = \sum_{i=1}^{N} \frac{\partial L}{\partial \hat{x}_i} \cdot (x_i - \mu) \cdot \left( -\frac{1}{2} \right) \cdot (\sigma^2 + \epsilon)^{-3/2}
$$

5. **对均值的梯度**：

$$
\frac{\partial L}{\partial \mu} = \sum_{i=1}^{N} \left( \frac{\partial L}{\partial \hat{x}_i} \cdot \left( - \frac{1}{\sqrt{\sigma^2 + \epsilon}} \right) \right) + \frac{\partial L}{\partial \sigma^2} \cdot \left( \frac{-2}{N} \sum_{i=1}^{N} (x_i - \mu) \right)
$$

6. **最终对 $x_i$ 的梯度**：

$$
\frac{\partial L}{\partial x_i} = \frac{\partial L}{\partial \hat{x}_i} \cdot \frac{1}{\sqrt{\sigma^2 + \epsilon}} + \frac{\partial L}{\partial \sigma^2} \cdot \frac{2(x_i - \mu)}{N} + \frac{\partial L}{\partial \mu} \cdot \frac{1}{N}
$$

---

## 代码

```python
def batchnorm_forward(x, gamma, beta, bn_param):
    mode = bn_param["mode"]
    eps = bn_param.get("eps", 1e-5)
    momentum = bn_param.get("momentum", 0.9)

    N, D = x.shape
    running_mean = bn_param.get("running_mean", np.zeros(D, dtype=x.dtype))
    running_var = bn_param.get("running_var", np.zeros(D, dtype=x.dtype))

    out, cache = None, None
    if mode == "train":
        # 均值
        sample_mean = np.mean(x, axis=0)
        # 方差
        sample_var = np.var(x, axis=0)
        # 标准化
        x_hat = (x - sample_mean) / np.sqrt(sample_var + eps)
        # 缩放和平移
        out = gamma * x_hat + beta

        cache = (x, gamma, beta, x_hat, sample_mean, sample_var, eps)

        running_mean = momentum * running_mean + (1 - momentum) * sample_mean
        running_var = momentum * running_var + (1 - momentum) * sample_var

    elif mode == "test":
        x_hat = (x - running_mean) / np.sqrt(running_var + eps)
        out = gamma * x_hat + beta

    else:
        raise ValueError('Invalid forward batchnorm mode "%s"' % mode)

    bn_param["running_mean"] = running_mean
    bn_param["running_var"] = running_var

    return out, cache


def batchnorm_backward(dout, cache):
    x, gamma, beta, x_hat, mean, var, eps = cache
    N, D = x.shape

    dbeta = np.sum(dout, axis=0)
    dgamma = np.sum(dout * x_hat, axis=0)

    # 反向传播链式法则
    dxhat = dout * gamma

    dvar = np.sum(dxhat * (x - mean) * -0.5 * (var + eps) ** (-1.5), axis=0)
    dmean = np.sum(dxhat * -1 / np.sqrt(var + eps), axis=0) + dvar * np.mean(-2 * (x - mean), axis=0)

    dx = dxhat / np.sqrt(var + eps) + dvar * 2 * (x - mean) / N + dmean / N

    return dx, dgamma, dbeta

```