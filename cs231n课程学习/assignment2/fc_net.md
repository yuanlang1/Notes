# Q1: Multi-Layer Fully Connected Neural Networks

## 多层神经网络前向与反向传播公式

### 一、符号说明

- 输入数据：\( X \in \mathbb{R}^{N \times D} \)
- 第 \( i \) 层权重与偏置：  
  \( W^{(i)} \in \mathbb{R}^{d_{i-1} \times d_i}, \quad b^{(i)} \in \mathbb{R}^{d_i} \)
- BatchNorm 可学习参数：  
  \( \gamma^{(i)}, \beta^{(i)} \in \mathbb{R}^{d_i} \)
- Dropout 保留概率：\( p \)
- 激活输入：\( a^{(0)} = X \)
- 总层数（不含 softmax loss）：\( L \)

---

###  二、前向传播（Forward）

#### 对于第 \( i = 1, 2, ..., L-1 \) 层：

1. **Affine 仿射变换：**
   \[
   z^{(i)} = a^{(i-1)} W^{(i)} + b^{(i)}
   \]

2. **Batch Normalization：**
   \[
   \mu^{(i)} = \frac{1}{N} \sum_{n=1}^N z_n^{(i)}, \quad
   \sigma^{2(i)} = \frac{1}{N} \sum_{n=1}^N (z_n^{(i)} - \mu^{(i)})^2
   \]
   \[
   \hat{z}^{(i)} = \frac{z^{(i)} - \mu^{(i)}}{\sqrt{\sigma^{2(i)} + \epsilon}}, \quad
   y^{(i)} = \gamma^{(i)} \cdot \hat{z}^{(i)} + \beta^{(i)}
   \]

3. **ReLU 激活：**
   \[
   a^{(i)} = \max(0, y^{(i)})
   \]

4. **Dropout（如果启用）：**
   \[
   \text{mask}^{(i)} \sim \text{Bernoulli}(p), \quad
   a^{(i)} = \frac{a^{(i)} \cdot \text{mask}^{(i)}}{p}
   \]

---

#### 第 \( L \) 层（最后一层，仅 affine）：

\[
\text{scores} = z^{(L)} = a^{(L-1)} W^{(L)} + b^{(L)}
\]

---

#### Softmax 损失函数：

\[
\text{softmax}_j = \frac{e^{s_j}}{\sum_k e^{s_k}}, \quad
\ell_i = -\log(\text{softmax}_{y_i}), \quad
\mathcal{L} = \frac{1}{N} \sum_{i=1}^N \ell_i + \frac{\lambda}{2} \sum_{i=1}^{L} \|W^{(i)}\|^2
\]

---

###  三、反向传播（Backward）

#### Softmax 输出梯度：

\[
\frac{\partial \mathcal{L}}{\partial s_{i,j}} = \frac{1}{N} \left( \text{softmax}_{i,j} - \mathbf{1}(j = y_i) \right)
\]

---

#### 第 \( L \) 层反向传播（Affine only）：

\[
\begin{aligned}
\frac{\partial \mathcal{L}}{\partial W^{(L)}} &= (a^{(L-1)})^T \cdot ds + \lambda W^{(L)} \\
\frac{\partial \mathcal{L}}{\partial b^{(L)}} &= \sum_{n=1}^N ds_n \\
\frac{\partial \mathcal{L}}{\partial a^{(L-1)}} &= ds \cdot (W^{(L)})^T
\end{aligned}
\]

---

#### 对于 \( i = L-1, ..., 1 \) 层反向传播：

1. **Dropout（若启用）：**
   \[
   da^{(i)} = \frac{da^{(i)} \cdot \text{mask}^{(i)}}{p}
   \]

2. **ReLU：**
   \[
   dy^{(i)} = da^{(i)} \cdot \mathbf{1}(y^{(i)} > 0)
   \]

3. **BatchNorm：**

   - 对参数：
     \[
     d\beta^{(i)} = \sum dy^{(i)}, \quad
     d\gamma^{(i)} = \sum (dy^{(i)} \cdot \hat{z}^{(i)})
     \]

   - 对归一化输入：
     \[
     d\hat{z}^{(i)} = dy^{(i)} \cdot \gamma^{(i)}
     \]

   - 中间变量求导：
     \[
     d\sigma^2 = \sum \left( d\hat{z}^{(i)} \cdot (z^{(i)} - \mu^{(i)}) \cdot -\frac{1}{2} (\sigma^{2(i)} + \epsilon)^{-3/2} \right)
     \]
     \[
     d\mu = \sum \left( d\hat{z}^{(i)} \cdot -\frac{1}{\sqrt{\sigma^{2(i)} + \epsilon}} \right) + d\sigma^2 \cdot \left( \frac{-2}{N} \sum (z^{(i)} - \mu^{(i)}) \right)
     \]
     \[
     dz^{(i)} = \frac{d\hat{z}^{(i)}}{\sqrt{\sigma^{2(i)} + \epsilon}} + d\sigma^2 \cdot \frac{2(z^{(i)} - \mu^{(i)})}{N} + \frac{d\mu}{N}
     \]

4. **Affine：**
   \[
   \begin{aligned}
   \frac{\partial \mathcal{L}}{\partial W^{(i)}} &= (a^{(i-1)})^T \cdot dz^{(i)} + \lambda W^{(i)} \\
   \frac{\partial \mathcal{L}}{\partial b^{(i)}} &= \sum dz^{(i)} \\
   \frac{\partial \mathcal{L}}{\partial a^{(i-1)}} &= dz^{(i)} \cdot (W^{(i)})^T
   \end{aligned}
   \]

---


## 代码

```python
# -*- coding = utf-8 -*-
import numpy as np

from cs231n.assignment2.utils.layer_utils import affine_bn_relu_forward, affine_relu_forward, affine_bn_relu_backward, \
    affine_relu_backward
from cs231n.assignment2.utils.layers import affine_forward, dropout_forward, softmax_loss, affine_backward, \
    dropout_backward


class FullyConnectNet(object):
    def __init__(self, hidden_dims, input_dims=32 * 32 * 3, num_classes=10,
                 dropout=0, use_batchnorm=False, reg=0.0,
                 weight_scale=1e-2, dtype=np.float32, seed=None):
        self.use_batchnorm = use_batchnorm
        self.use_dropout = dropout > 0
        self.reg = reg
        self.num_layers = 1 + len(hidden_dims)
        self.dtype = dtype
        self.params = {}

        self.params["W1"] = np.random.randn(input_dims, hidden_dims[0]) * weight_scale
        self.params["b1"] = np.zeros(hidden_dims[0])

        for i in range(self.num_layers - 2):
            self.params["W" + str(i + 2)] = np.random.randn(hidden_dims[i], hidden_dims[i + 1]) * weight_scale
            self.params["b" + str(i + 2)] = np.zeros(hidden_dims[i + 1])

            if self.use_batchnorm:
                self.params["gamma" + str(i + 1)] = np.ones(hidden_dims[i])
                self.params["beta" + str(i + 1)] = np.ones(hidden_dims[i])

        if self.use_batchnorm:
            self.params["gamma" + str(self.num_layers - 1)] = np.ones(hidden_dims[-1])
            self.params["beta" + str(self.num_layers - 1)] = np.ones(hidden_dims[-1])
        self.params["W" + str(self.num_layers)] = np.random.randn(hidden_dims[-1], num_classes)
        self.params["b" + str(self.num_layers)] = np.random.randn(num_classes)

        self.dropout_param = {}
        if self.use_dropout:
            self.dropout_param = {"mode": "train", "p": dropout}
            if seed is not None:
                self.dropout_param["seed"] = seed

        self.bn_param = {}
        if self.use_batchnorm:
            self.bn_params = [{"mode": "train"} for i in range(self.num_layers - 1)]

        for k, v in self.params.items():
            self.params[k] = v.astype(dtype)

    def loss(self, X, y=None):
        X = X.astype(self.dtype)
        mode = "test" if y is None else "train"

        if self.dropout_param is not None:
            self.dropout_param["mode"] = mode
        if self.use_batchnorm:
            for bn_param in self.bn_params:
                bn_param[mode] = mode

        scores = None

        hidden_layers, caches = range(self.num_layers + 1), range(self.num_layers)
        dp_caches = range(self.num_layers - 1)
        hidden_layers[0] = X
        for i in range(self.num_layers):
            W, b = self.params["W" + str(i + 1)], self.params["b" + str(i + 1)]
            if i == self.num_layers - 1:
                hidden_layers[i + 1], caches[i] = affine_forward(hidden_layers[i], W, b)

            else:
                if self.use_batchnorm:
                    gamma, beta = self.params["gamma" + str(i + 1)], self.params["beta" + str(i + 1)]
                    hidden_layers[i + 1], caches[i] = affine_bn_relu_forward(hidden_layers[i], W, b, gamma, beta, self.bn_params[i])
                else:
                    hidden_layers[i + 1], caches[i] = affine_relu_forward(hidden_layers[i], W, b)
                if self.use_dropout:
                    hidden_layers[i + 1], dp_caches[i] = dropout_forward(hidden_layers[i + 1], self.dropout_param)

        scores = hidden_layers[self.num_layers]

        if mode == "test":
            return scores

        loss, grads = 0.0, {}

        loss, dscores = softmax_loss(scores, y)
        dhiddens = range(self.num_layers + 1)
        dhiddens[self.num_layers] = dscores

        for i in range(self.num_layers, 0, -1):
            if i == self.num_layers:
                dhiddens[i - 1], grads["W" + str(i)], grads["b" + str(i)] = affine_backward(dhiddens[i], caches[i - 1])
            else:
                if self.use_dropout:
                    dhiddens[i] = dropout_backward(dhiddens[i], dp_caches[i - 1])
                if self.use_batchnorm:
                    dx, dw, db, dgamma, dbeta = affine_bn_relu_backward(dhiddens[i], caches[i - 1])
                    dhiddens[i - 1], grads["W" + str(i)], grads["b" + str(i)] = dx, dw, db
                    grads["gamma" + str(i)], grads["beta" + str(i)] = dgamma, dbeta
                else:
                    dx, dw, db = affine_relu_backward(dhiddens[i], caches[i - 1])
                    dhiddens[i - 1], grads["W" + str(i)], grads["b" + str(i)] = dx, dw, db

            loss += 0.5 * self.reg * np.sum(self.params["W" + str(i)] ** 2)
            grads["W" + str(i)] += self.reg * self.params["W" + str(i)]

        return loss, grads
```
