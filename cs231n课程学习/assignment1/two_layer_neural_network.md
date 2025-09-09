# Q4: Two-Layer Neural Network

# Two-Layer Neural Network：公式推导

假设有：
- 输入矩阵 \(X\in\mathbb{R}^{N\times D}\)，共 \(N\) 个样本，每个样本 \(D\) 维。
- 第一层参数：权重 \(W_1\in\mathbb{R}^{D\times H}\)，偏置 \(b_1\in\mathbb{R}^H\)。
- 第二层参数：权重 \(W_2\in\mathbb{R}^{H\times C}\)，偏置 \(b_2\in\mathbb{R}^C\)。
- 真值标签 \(y\in\{0,\dots,C-1\}^N\)。
- 正则化系数 \(\lambda\)。

---

## 1. Forward Pass

1. **隐藏层线性变换**  
   \[
   Z = XW_1 + b_1,\quad Z\in\mathbb{R}^{N\times H}
   \]

2. **ReLU 激活**  
   \[
   A = \mathrm{ReLU}(Z) = \max(0,\,Z)
   \]

3. **输出层线性变换（logits）**  
   \[
   S = AW_2 + b_2,\quad S\in\mathbb{R}^{N\times C}
   \]

4. **Softmax 概率**  
   为数值稳定，先平移：  
   \[
   \tilde S_{i,j} = S_{i,j} - \max_k S_{i,k}
   \]  
   再归一化：  
   \[
   P_{i,j} = \frac{e^{\tilde S_{i,j}}}{\sum_{k=1}^C e^{\tilde S_{i,k}}}
   \]

---

## 2. Loss Function

**交叉熵损失 + L2 正则：**  
\[
\mathcal{L}
= -\frac{1}{N}\sum_{i=1}^N \log\bigl(P_{i,y_i}\bigr)
\;+\;\frac{\lambda}{2}\Bigl(\|W_1\|_F^2 + \|W_2\|_F^2\Bigr)
\]

---

## 3. Backward Pass（梯度）

1. **对 logits 的梯度**  
   定义误差矩阵  
   \[
   E_{i,j} = P_{i,j} - \mathbf{1}_{\{j=y_i\}},
   \quad \mathbf{1}_{\{j=y_i\}}=\begin{cases}1,&j=y_i\\0,&\text{otherwise}\end{cases}
   \]  
   则  
   \[
   \frac{\partial\mathcal{L}}{\partial S} = \frac{1}{N}E.
   \]

2. **输出层参数梯度**  
   \[
   \frac{\partial\mathcal{L}}{\partial W_2}
   = A^\top \Bigl(\tfrac{1}{N}E\Bigr) \;+\;\lambda W_2,
   \quad
   \frac{\partial\mathcal{L}}{\partial b_2}
   = \sum_{i=1}^N \frac{1}{N}E_{i,\ast}.
   \]

3. **隐藏层反向传播**  
   \[
   \frac{\partial\mathcal{L}}{\partial A}
   = \frac{1}{N}E\,W_2^\top,\quad
   \frac{\partial\mathcal{L}}{\partial Z}
   = \frac{\partial\mathcal{L}}{\partial A}\;\odot\;\mathbf{1}_{\{Z>0\}}.
   \]

4. **输入层参数梯度**  
   \[
   \frac{\partial\mathcal{L}}{\partial W_1}
   = X^\top \Bigl(\tfrac{\partial\mathcal{L}}{\partial Z}\Bigr)
   \;+\;\lambda W_1,
   \quad
   \frac{\partial\mathcal{L}}{\partial b_1}
   = \sum_{i=1}^N \bigl[\tfrac{\partial\mathcal{L}}{\partial Z}\bigr]_{i,\ast}.
   \]


---

## 代码

```python
# -*- coding = utf-8 -*-
import numpy as np
import matplotlib.pyplot as plt

from cs231n.data_utils import load_CIFAR10


class TwoLayerNet(object):
    def __init__(self, input_size, hidden_size, ouput_size, std=1e-4):
        self.params = {}
        self.params["W1"] = std * np.random.randn(input_size, hidden_size)
        self.params["b1"] = np.zeros(hidden_size)
        self.params["W2"] = std * np.random.randn(hidden_size, ouput_size)
        self.params["b2"] = np.zeros(ouput_size)

    def loss(self, X, y=None, reg=0.0):
        W1, b1 = self.params["W1"], self.params["b1"]
        W2, b2 = self.params["W2"], self.params["b2"]

        # forward pass
        hidden = np.maximum(0, X.dot(W1) + b1)
        scores = hidden.dot(W2) + b2

        if y is None:
            return scores

        # Softmax loss
        scores -= np.max(scores, axis=1, keepdims=True)
        exp_scores = np.exp(scores)
        probs = exp_scores / np.sum(exp_scores, axis=1, keepdims=True)

        N = X.shape[0]
        correct_log_probs = -np.log(probs[np.arange(N), y])
        data_loss = np.sum(correct_log_probs) / N
        reg_loss = 0.5 * reg * (np.sum(W1 * W1) + np.sum(W2 * W2))
        loss = data_loss + reg_loss

        # backward pass
        grads = {}
        dscores = probs
        dscores[np.arange(N), y] -= 1
        dscores /= N

        grads["W2"] = hidden.T.dot(dscores) + reg * W2
        grads["b2"] = np.sum(dscores, axis=0)

        dhidden = dscores.dot(W2.T)
        dhidden[hidden <= 0] = 0

        grads["W1"] = X.T.dot(dhidden) + reg * W1
        grads["b1"] = np.sum(dhidden, axis=0)

        return loss, grads

    def train(self, X, y, X_val, y_val,
              learning_rate=1e-3, learning_rate_decay=0.95,
              reg=1e-5, num_iters=100,
              batch_size=200, verbose=False):
        num_train = X.shape[0]
        iterations_per_epoch = max(num_train // batch_size, 1)

        loss_history = []
        train_acc_history = []
        val_acc_history = []

        for it in range(num_iters):
            batch_indices = np.random.choice(num_train, batch_size)
            X_batch = X[batch_indices]
            y_batch = y[batch_indices]

            loss, grads = self.loss(X_batch, y_batch, reg)
            loss_history.append(loss)

            for param in self.params:
                self.params[param] -= learning_rate * grads[param]

            if verbose and it % 100 == 0:
                print(f"Iteration {it}/{num_iters}: loss {loss:.4f}")

            if it % iterations_per_epoch == 0:
                train_acc = (self.predict(X_batch) == y_batch).mean()
                val_acc = (self.predict(X_val) == y_val).mean()
                train_acc_history.append(train_acc)
                val_acc_history.append(val_acc)

                learning_rate *= learning_rate_decay

        return {
            "loss_history": loss_history,
            "train_acc_history": train_acc_history,
            "val_acc_history": val_acc_history
        }

    def predict(self, X):
        hidden = np.maximum(0, X.dot(self.params["W1"]) + self.params["b1"])
        scores = hidden.dot(self.params["W2"]) + self.params["b2"]
        y_pred = np.argmax(scores, axis=1)
        return y_pred


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

    # Normalize the data: subtract the mean image
    mean_image = np.mean(X_train, axis=0)
    X_train -= mean_image
    X_val -= mean_image
    X_test -= mean_image

    return X_train, y_train, X_val, y_val, X_test, y_test


X_train, y_train, X_val, y_val, X_test, y_test = get_CIFAR10_data()
input_size = 32*32*3
hidden_size = 50
num_classes = 10
net = TwoLayerNet(input_size, hidden_size, num_classes)

stats = net.train(X_train, y_train, X_val, y_val,
                  num_iters=1000, batch_size=400,
                  learning_rate=1e-3, learning_rate_decay=0.95,
                  reg=0.5, verbose=True)

val_acc = (net.predict(X_val) == y_val).mean()
print(f"Validation accuracy:{val_acc}")


# Plot the loss function and train / validation accuracies
plt.subplot(2, 1, 1)
plt.plot(stats['loss_history'])
plt.title('Loss history')
plt.xlabel('Iteration')
plt.ylabel('Loss')

plt.subplot(2, 1, 2)
plt.plot(stats['train_acc_history'], label='train')
plt.plot(stats['val_acc_history'], label='val')
plt.title('Classification accuracy history')
plt.xlabel('Epoch')
plt.ylabel('Clasification accuracy')
plt.show()

```