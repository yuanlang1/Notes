# Q2: Training a Support Vector Machine

## 多类 SVM 损失函数与梯度

**参数与符号**  
- 输入数据矩阵：\(X \in \mathbb{R}^{N\times D}\)  
- 标签向量：\(y \in \{0,1,\dots,C-1\}^N\)  
- 权重矩阵：\(W \in \mathbb{R}^{D\times C}\)  
- 正则化强度：\(\lambda\)  
- 间隔常数：\(\Delta = 1\)

---

### 1. 前向传播：计算得分  
\[
s_i = X_i W
\quad\Longrightarrow\quad
s_{i,j} = X_i \cdot W_{:,j} 
\]

---

### 2. 计算 Margin  
对于每个样本 \(i\)、每个类别 \(j\)：  
\[
\text{margin}_{i,j}
= \max\bigl(0,\;s_{i,j} - s_{i,y_i} + \Delta\bigr)
\]
并令正确类的 margin 为 0：  
\[
\text{margin}_{i,y_i} = 0
\]

---

### 3. 损失函数  
#### 数据损失  
\[
L_{\text{data}}
= \frac{1}{N}\sum_{i=1}^N\sum_{j=1}^C \text{margin}_{i,j}
\]
#### 正则化损失  
\[
L_{\text{reg}}
= \frac{\lambda}{2}\sum_{d=1}^D\sum_{c=1}^C W_{d,c}^2
\]
#### 总损失  
\[
L = L_{\text{data}} + L_{\text{reg}}
\]

---

### 4. 反向传播：梯度公式  
定义指示函数  
\[
\mathbf{1}_{i,j} =
\begin{cases}
1, & \text{if }\text{margin}_{i,j}>0,\\
0, & \text{otherwise}
\end{cases}
\]
构造计数矩阵  
\[
C_{i,j} = \mathbf{1}_{i,j},
\quad
C_{i,y_i} = -\sum_{j\ne y_i} \mathbf{1}_{i,j}.
\]
则梯度为  
\[
\frac{\partial L}{\partial W}
= \frac{1}{N} X^\top C \;+\; \lambda W.
\]

---

## 代码

```python
import numpy as np

from cs231n.data_utils import load_CIFAR10


def svm_loss(W, X, y, reg):
    loss = 0.0
    dW = np.zeros(W.shape)
    N = X.shape[0]
    # 分数
    scores = X.dot(W)

    # 计算margin
    margin = scores - scores[range(0, N), y].reshape(-1, 1) + 1
    # 正确项为0
    margin[range(N), y] = 0
    # > 0
    margin = (margin > 0) * margin

    # 损失
    loss += margin.sum() / N
    # 正则项损失
    loss += 0.5 * reg * np.sum(W * W)  # L2正则项
    
    # 梯度计算
    # 错误分类
    counts = (margin > 0).astype(int)
    # 正确分类
    counts[range(N), y] = -np.sum(counts, axis=1)
    # 正则项
    dW += np.dot(X.T, counts) / N + reg * W

    return loss, dW


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


X_train, y_train, X_val, y_val, X_test, y_test, X_dev, y_dev = get_CIFAR10_data()

W = np.random.randn(3073, 10) * 0.0001
loss, grad = svm_loss(W, X_dev, y_dev, 0.00001)
print(f"loss: {loss}")

```