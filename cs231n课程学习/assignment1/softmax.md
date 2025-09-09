# Q3: Implement a Softmax classifier

--- 

## Softmax Loss 公式

对于每个样本 \( i \)，其得分向量为：
\[
f_i = X_i \cdot W
\]

Softmax 概率：
\[
p_{i,j} = \frac{e^{f_{i,j}}}{\sum_k e^{f_{i,k}}}
\]

交叉熵损失（不含正则）：
\[
L_i = -\log(p_{i, y_i}) = \log \sum_k e^{f_{i,k}} - f_{i, y_i}
\]

总损失：
\[
L = \frac{1}{N} \sum_{i=1}^N L_i + \frac{1}{2} \lambda \sum W^2
\]

梯度：
\[
\frac{\partial L}{\partial W} = \frac{1}{N} X^T (P - Y) + \lambda W
\]
其中：
- \( P \) 是 Softmax 概率矩阵；
- \( Y \) 是 one-hot 编码标签。

---

## 代码

```python
# -*- coding = utf-8 -*-
import random

import matplotlib
import numpy as np
from cs231n.data_utils import load_CIFAR10
import matplotlib.pyplot as plt


def get_CIFAR10_data(num_training=49000, num_validation=1000, num_test=1000, num_dev=500):
    """
    Load the CIFAR-10 dataset from disk and perform preprocessing to prepare
    it for the linear classifier. These are the same steps as we used for the
    SVM, but condensed to a single function.
    """
    # Load the raw CIFAR-10 data
    cifar10_dir = 'cifar-10-python/cifar-10-batches-py'
    X_train, y_train, X_test, y_test = load_CIFAR10(cifar10_dir)

    # subsample the data
    mask = range(num_training, num_training + num_validation)
    X_val = X_train[mask]
    y_val = y_train[mask]
    mask = range(num_training)
    X_train = X_train[mask]
    y_train = y_train[mask]
    mask = range(num_test)
    X_test = X_test[mask]
    y_test = y_test[mask]
    mask = np.random.choice(num_training, num_dev, replace=False)
    X_dev = X_train[mask]
    y_dev = y_train[mask]

    # Preprocessing: reshape the image data into rows
    X_train = np.reshape(X_train, (X_train.shape[0], -1))
    X_val = np.reshape(X_val, (X_val.shape[0], -1))
    X_test = np.reshape(X_test, (X_test.shape[0], -1))
    X_dev = np.reshape(X_dev, (X_dev.shape[0], -1))

    # Normalize the data: subtract the mean image
    mean_image = np.mean(X_train, axis=0)
    X_train -= mean_image
    X_val -= mean_image
    X_test -= mean_image
    X_dev -= mean_image

    # add bias dimension and transform into columns
    X_train = np.hstack([X_train, np.ones((X_train.shape[0], 1))])
    X_val = np.hstack([X_val, np.ones((X_val.shape[0], 1))])
    X_test = np.hstack([X_test, np.ones((X_test.shape[0], 1))])
    X_dev = np.hstack([X_dev, np.ones((X_dev.shape[0], 1))])

    return X_train, y_train, X_val, y_val, X_test, y_test, X_dev, y_dev


def softmax_loss(W, X, y, reg):
    loss = 0.0
    dW = np.zeros_like(W)
    N = X.shape[0]

    # 得分
    f = np.dot(X, W)
    f -= f.max(axis=1).reshape(N, 1)

    # 计算softmax概率
    s = np.exp(f).sum(axis=1)

    # 计算交叉熵损失
    loss = np.log(s).sum() - f[range(N), y].sum()
    loss = loss / N + 0.5 * reg * np.sum(W * W)  # L2正则化

    # 计算梯度
    counts = np.exp(f) / s.reshape(N, 1)
    counts[range(N), y] -= 1
    dW = np.dot(X.T, counts)
    dW = dW / N + reg * W

    return loss, dW


X_train, y_train, X_val, y_val, X_test, y_test, X_dev, y_dev = get_CIFAR10_data()

W = np.random.randn(3073, 10) * 0.0001
loss, grad = softmax_loss(W, X_dev, y_dev, 0.0)

print(loss)

```