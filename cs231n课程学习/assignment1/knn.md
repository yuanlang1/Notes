# Q1：  k-Nearest Neighbor classifier

## KNN 分类器核心公式

1. **距离度量**  
   - **L1 距离（曼哈顿距离）**  
     $$d(x, x_i) = \sum_{j=1}^D \bigl|x_j - x_{ij}\bigr| = \left\|x-x_{i}\right\|_{1}$$  
   - **L2 距离（欧几里得距离）**  
     $$d(x, x_i) = \sqrt{\sum_{j=1}^D \bigl(x_j - x_{ij}\bigr)^2} =\left\|x-x_{i}\right\|_{2}$$  
     （排序时可省略开根号，因为单调性不变）

2. **寻找 \(k\) 个最近邻**  
   计算所有训练样本的距离向量  
   \(\{d(x, x_i)\}_{i=1}^N\)，取最小的 \(k\) 个索引：  
   $$
   \mathcal{N}_k(x)
   = \operatorname{argsort}\bigl(\{d(x, x_i)\}\bigr)_{1:k}
   $$

3. **多数投票决定类别**  
   令这 \(k\) 个邻居的标签为 \(\{y_i \mid i \in \mathcal{N}_k(x)\}\)，则预测类别：  
   $$
   \hat{y}
   = \arg\max_{c}\sum_{i \in \mathcal{N}_k(x)} \mathbf{1}\bigl(y_i = c\bigr)
   $$  
   其中 \(\mathbf{1}(\cdot)\) 是指示函数，满足  
   \[
     \mathbf{1}(y_i = c) =
     \begin{cases}
       1, & y_i = c,\\
       0, & \text{otherwise}.
     \end{cases}
   \]


```python
# -*- coding = utf-8 -*-
import numpy as np

from cs231n.data_utils import load_CIFAR10


class KNearestNeighbor(object):
    def __init__(self):
        pass

    def train(self, X, y):
        self.Xtr = X
        self.ytr = y

    def predict(self, X, k=1):
        num_test = X.shape[0]
        Ypred = np.zeros(num_test)

        for i in range(num_test):
            # L1距离
            dists = np.sum(np.abs(self.Xtr - X[i, :]), axis=1)
            # 拍序，取前k个最小距离对应的标签
            closest_y = self.ytr[np.argsort(dists)[0:k]]
            # 选取出现次数最多的标签
            Ypred[i] = np.bincount(closest_y).argmax()

        return Ypred


Xtr, Ytr, Xte, Yte = load_CIFAR10("cifar-10-python/cifar-10-batches-py")
Xtr_rows = Xtr.reshape(Xtr.shape[0], 32 * 32 * 3)
Xte_rows = Xte.reshape(Xte.shape[0], 32 * 32 * 3)

Xval_rows = Xtr_rows[:1000, :]
Yval = Ytr[:1000]
Xtr_rows = Xtr_rows[1000:, :]
Ytr = Ytr[1000:]

validation_accuracies = []
for k in [1, 3, 5, 10, 20, 50, 100]:
    nn = KNearestNeighbor()
    nn.train(Xtr_rows, Ytr)
    Yval_predict = nn.predict(Xval_rows, k=k)
    acc = np.mean(Yval_predict == Yval)
    print('accuracy : %f' % (acc,))

    validation_accuracies.append((k, acc))
```

