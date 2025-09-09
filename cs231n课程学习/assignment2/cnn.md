# Convolutional Neural Networks

## 三层卷积神经网络（ThreeLayerConvNet）数学公式说明

---

### 1. 网络结构

输入尺寸为 \( (N, C, H, W) \)，其中 \(N\) 是批量大小。

- 第一层：卷积层 + ReLU + 2x2 最大池化  
- 第二层：全连接层 + ReLU  
- 第三层：全连接层（输出层，无激活）

---

### 2. 参数定义

- \( W_1 \in \mathbb{R}^{F \times C \times HH \times WW} \)，卷积核权重  
- \( b_1 \in \mathbb{R}^F \)，卷积偏置  
- \( W_2 \in \mathbb{R}^{D \times H} \)，第一全连接层权重，\( D = F \times \frac{H}{2} \times \frac{W}{2} \)  
- \( b_2 \in \mathbb{R}^H \)，第一全连接层偏置  
- \( W_3 \in \mathbb{R}^{H \times C} \)，第二全连接层权重（输出层）  
- \( b_3 \in \mathbb{R}^C \)，第二全连接层偏置

---

### 3. 前向传播公式

#### 3.1 卷积 + ReLU + 最大池化

\[
Z^{(1)} = X * W_1 + b_1
\]

\[
A^{(1)} = \text{ReLU}(Z^{(1)}) = \max(0, Z^{(1)})
\]

\[
P^{(1)} = \text{MaxPool}(A^{(1)}, \text{pool\_size}=2, \text{stride}=2)
\]

---

#### 3.2 展平

将池化层输出展平成二维矩阵：

\[
P^{(1)}_{\text{flat}} \in \mathbb{R}^{N \times D}, \quad D = F \times \frac{H}{2} \times \frac{W}{2}
\]

---

#### 3.3 第一全连接层 + ReLU

\[
Z^{(2)} = P^{(1)}_{\text{flat}} W_2 + b_2
\]

\[
A^{(2)} = \text{ReLU}(Z^{(2)}) = \max(0, Z^{(2)})
\]

---

#### 3.4 第二全连接层（输出层）

\[
S = A^{(2)} W_3 + b_3
\]

---

### 4. Softmax 损失

给定标签 \( y \)，损失函数为：

\[
L = -\frac{1}{N} \sum_{i=1}^N \log \frac{e^{S_{i, y_i}}}{\sum_{j} e^{S_{i, j}}} + \frac{\lambda}{2} \left( \|W_1\|_F^2 + \|W_2\|_F^2 + \|W_3\|_F^2 \right)
\]

其中 \( \lambda \) 为正则化强度。

---

### 5. 反向传播公式

- **输出层误差：**

\[
\frac{\partial L}{\partial S_{ij}} = \frac{1}{N}(p_{ij} - \mathbf{1}_{j=y_i})
\]

其中

\[
p_{ij} = \frac{e^{S_{ij}}}{\sum_k e^{S_{ik}}}
\]

- **第二全连接层梯度：**

\[
\frac{\partial L}{\partial W_3} = (A^{(2)})^T \frac{\partial L}{\partial S} + \lambda W_3
\]

\[
\frac{\partial L}{\partial b_3} = \sum_i \frac{\partial L}{\partial S_i}
\]

- **第一全连接层梯度（包括 ReLU 反向传播）：**

\[
\delta^{(2)} = \left(\frac{\partial L}{\partial S} W_3^T \right) \odot \mathbf{1}_{Z^{(2)}>0}
\]

\[
\frac{\partial L}{\partial W_2} = (P^{(1)}_{\text{flat}})^T \delta^{(2)} + \lambda W_2
\]

\[
\frac{\partial L}{\partial b_2} = \sum_i \delta^{(2)}_i
\]

- **卷积层梯度（通过 conv_relu_pool_backward 计算）：**

\[
\delta^{(1)} = \text{backprop through pool, ReLU, conv}
\]

\[
\frac{\partial L}{\partial W_1}, \quad \frac{\partial L}{\partial b_1}
\]

---

```text
输入 X
  ↓ 卷积 W1, b1 + ReLU + MaxPool
  ↓ Flatten
  ↓ 全连接 W2, b2 + ReLU
  ↓ 全连接 W3, b3
  ↓ Softmax + 损失
```

---

## 代码

```python
# -*- coding = utf-8 -*-
import numpy as np
from matplotlib import pyplot as plt

from cs231n.assignment2.utils.layer_utils import conv_relu_pool_forward, affine_relu_forward, affine_relu_backward, \
    conv_relu_pool_backward
from cs231n.assignment2.utils.layers import affine_forward, softmax_loss, affine_backward
from cs231n.assignment2.utils.solver import Solver
from cs231n.data_utils import get_CIFAR10_data


class ThreeLayerConvNet(object):
    def __init__(self, input_dims=(3, 32, 32), num_filters=32, filter_size=7,
                 hidden_dim=100, num_classes=10, weight_scale=1e-3, reg=0.0,
                 dtype=np.float32):
        self.params = {}
        self.reg = reg
        self.dtype = dtype

        C, H, W = input_dims
        F, HH, WW = num_filters, filter_size, filter_size
        self.params["W1"] = weight_scale * np.random.randn(F, C, HH, WW)
        self.params["W2"] = weight_scale * np.random.randn(F * H // 2 * W // 2, hidden_dim)
        self.params["W3"] = weight_scale * np.random.randn(hidden_dim, num_classes)

        self.params["b1"] = np.zeros(F)
        self.params["b2"] = np.zeros(hidden_dim)
        self.params["b3"] = np.zeros(num_classes)

        for k, v in self.params.items():
            self.params[k] = v.astype(dtype)

    def loss(self, X, y=None):
        W1, b1 = self.params["W1"], self.params["b1"]
        W2, b2 = self.params["W2"], self.params["b2"]
        W3, b3 = self.params["W3"], self.params["b3"]

        filter_size = W1.shape[2]
        conv_param = {"stride": 1, "pad": (filter_size - 1) // 2}
        pool_param = {"pool_height": 2, "pool_width": 2, "stride": 2}

        scores = None

        pool_out, pool_cache = conv_relu_pool_forward(X, W1, b1, conv_param, pool_param)
        affine_out, affine_cache = affine_relu_forward(pool_out, W2, b2)
        scores, cache = affine_forward(affine_out, W3, b3)

        if y is None:
            return scores

        loss, grads = 0, {}
        loss, dscores = softmax_loss(scores, y)
        daffine, grads["W3"], grads["b3"] = affine_backward(dscores, cache)
        dpool, grads["W2"], grads["b2"] = affine_relu_backward(daffine, affine_cache)
        dx, grads["W1"], grads["b1"] = conv_relu_pool_backward(dpool, pool_cache)

        loss += 0.5 * self.reg * (np.sum(W1 ** 2) + np.sum(W2 ** 2) + np.sum(W3 ** 2))
        grads["W1"] += self.reg * W1
        grads["W2"] += self.reg * W2
        grads["W3"] += self.reg * W3

        return loss, grads


data = get_CIFAR10_data()

# num_train = 100
# small_data = {
#     "X_train": data["X_train"][:num_train],
#     "y_train": data["y_train"][:num_train],
#     "X_val": data["X_val"],
#     "y_val": data["y_val"]
# }
#
# model = ThreeLayerConvNet(weight_scale=1e-2)
#
# solver = Solver(model, small_data,
#                 num_epochs=20, batch_size=50,
#                 update_rule="adam",
#                 optim_config={
#                     "learning_rate": 1e-4,
#                 },
#                 verbose=True, print_every=1)
#
# solver.train()
#
# plt.subplot(2, 1, 1)
# plt.plot(solver.loss_history, 'o')
# plt.xlabel('iteration')
# plt.ylabel('loss')
#
# plt.subplot(2, 1, 2)
# plt.plot(solver.train_acc_history, '-o')
# plt.plot(solver.val_acc_history, '-o')
# plt.legend(['train', 'val'], loc='upper left')
# plt.xlabel('epoch')
# plt.ylabel('accuracy')
# plt.show()

model = ThreeLayerConvNet(weight_scale=0.001, hidden_dim=500, reg=0.001)

solver = Solver(model, data,
                num_epochs=1, batch_size=50,
                update_rule="adam",
                optim_config={
                    "learning_rate":1e-4,
                },
                verbose=True, print_every=20)

solver.train()

```