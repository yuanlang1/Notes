# GAN（生成对抗网络）

---

## 概念

### 举例

举例两个角色：

- 造假师(`Generator， G`)：“伪造”数据，让假数据看起来像真的。
- 鉴定师(`Discriminator, D`)：“分辨”数据是真是假。

训练过程就像：

- 造假者想骗过鉴定师，鉴定师想抓住造假者。
随着时间推移，造假者越来越会造，鉴定师也越来越会分辨。
最终，假货几乎能以假乱真。

### 原理

#### 架构

`GAN`包含两个主要部分：

- `G`生成器：输入一个随机噪声`z`，输出生成的样本(如图片)
- `D`判别器：输入一个样本(真数据/`G`生成的数据），输出一个概率(样本为真的概率)

#### 对抗过程

1. 固定`G`，训练`D`

- 给`D`真实数据，希望输出接近1
- 给`G`生成数据，希望输出接近0
- 目标：让`D`学会分辨真假

2. 固定`D`，训练`G`

- 生成数据喂给`D`
- 希望`D`把它判断为1（骗过`D`）
- 目标：让`G`生成得更像真数据

---

## 数学公式

`GAN`的目标函数：

\[
\min_G \max_D V(D, G) = \mathbb{E}_{x \sim p_{data}(x)} [\log D(x)] + \mathbb{E}_{z \sim p_z(z)} [\log (1 - D(G(z)))]
\]

- 判别器 \(D\) 尽量正确区分真假样本，使 \(D(x)\) 趋近于1， \(D(G(z))\) 趋近于0。
- 生成器 \(G\) 尽量“骗过”判别器，使 \(D(G(z))\) 趋近于1。

损失函数：

损失函数为二分类交叉熵损失：

- 判别器损失：

\[
L_D = -\mathbb{E}_{x \sim p_{data}}[\log D(x)] - \mathbb{E}_{z \sim p_z}[\log (1 - D(G(z)))]
\]

- 生成器损失：

\[
L_G = -\mathbb{E}_{z \sim p_z}[\log D(G(z))]
\]

---

## 训练步骤

1. 从真实数据中采样一批样本，训练判别器，让它输出接近1。
2. 从随机噪声采样，生成假样本，训练判别器，让它输出接近0。
3. 固定判别器，训练生成器，提升生成假样本“骗过”判别器的能力。

---

## 代码

```python
import os

import matplotlib.pyplot as plt
import torch
from torch import nn, optim
from torch.utils.data import DataLoader
from torchvision import transforms, datasets
import torchvision.utils as vutils


class DC_Discriminator(nn.Module):
    def __init__(self):
        super().__init__()
        self.model = nn.Sequential(
            nn.Unflatten(1, (1, 28, 28)),
            nn.Conv2d(1, 32, 5, stride=1),
            nn.LeakyReLU(0.2, inplace=True),
            nn.MaxPool2d(2, 2),
            nn.Conv2d(32, 64, 5, stride=1),
            nn.LeakyReLU(0.2, inplace=True),
            nn.MaxPool2d(2, 2),
            nn.Flatten(),
            nn.Linear(4 * 4 * 64, 4 * 4 * 64),
            nn.LeakyReLU(0.2, inplace=True),
            nn.Linear(4 * 4 * 64, 1)
        )

    def forward(self, x):
        return self.model(x)


class DC_Generator(nn.Module):
    def __init__(self, noise_dim=100):
        super().__init__()
        self.model = nn.Sequential(
            nn.Linear(noise_dim, 1024),
            nn.ReLU(True),
            nn.BatchNorm1d(1024),
            nn.Linear(1024, 7 * 7 * 128),
            nn.ReLU(True),
            nn.BatchNorm1d(7 * 7 * 128),
            nn.Unflatten(1, (128, 7, 7)),
            nn.ConvTranspose2d(128, 64, 4, stride=2, padding=1),
            nn.ReLU(True),
            nn.BatchNorm2d(64),
            nn.ConvTranspose2d(64, 1, 4, stride=2, padding=1),
            nn.Tanh(),
            nn.Flatten()
        )

    def forward(self, z):
        return self.model(z)


batch_size = 128
lr = 1e-4
epochs = 20
noise_dim = 100
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

save_dir = "gan_images"
os.makedirs(save_dir, exist_ok=True)

transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.5, ), (0.5, ))
])

train_loader = DataLoader(
    datasets.MNIST(root="./data", train=True, download=True, transform=transform),
    batch_size=batch_size, shuffle=True
)

D = DC_Discriminator().to(device)
G = DC_Generator().to(device)

criterion = nn.BCEWithLogitsLoss()
optimizer_D = optim.Adam(D.parameters(), lr=lr, betas=(0.5, 0.999))
optimizer_G = optim.Adam(G.parameters(), lr=lr, betas=(0.5, 0.999))

fixed_noise = torch.randn(64, noise_dim, device=device)

epoch_images = []

for epoch in range(epochs):
    for real_imgs, _ in train_loader:
        real_imgs = real_imgs.view(real_imgs.size(0), -1).to(device)
        batch_size_curr = real_imgs.size(0)

        real_labels = torch.ones(batch_size_curr, 1, device=device)
        fake_labels = torch.zeros(batch_size_curr, 1, device=device)

        # 判别器
        optimizer_D.zero_grad()
        out_real = D(real_imgs)
        loss_real = criterion(out_real, real_labels)

        noise = torch.randn(batch_size_curr, noise_dim, device=device)
        fake_imgs = G(noise).detach()
        out_fake = D(fake_imgs)
        loss_fake = criterion(out_fake, fake_labels)

        loss_D = loss_real + loss_fake
        loss_D.backward()
        optimizer_D.step()

        # 生成器
        optimizer_G.zero_grad()
        noise = torch.randn(batch_size_curr, noise_dim, device=device)
        fake_imgs = G(noise)
        out_fake = D(fake_imgs)
        loss_G = criterion(out_fake, real_labels)
        loss_G.backward()
        optimizer_G.step()

    print(f"Epoch [{epoch + 1}/{epochs}] Loss_D: {loss_D.item():.4f} Loss_G: {loss_G.item():0.4f}")

    with torch.no_grad():
        samples = G(fixed_noise).view(-1, 1, 28, 28)
        grid = vutils.make_grid(samples, nrow=8, normalize=True, value_range=(-1, 1))

        plt.figure(figsize=(8, 8))
        plt.imshow(grid.permute(1, 2, 0).cpu())
        plt.axis("off")
        plt.title(f"Epoch {epoch + 1}", fontsize=16)
        plt.savefig(f"{save_dir}/epoch_{epoch + 1:03d}.png")
        plt.close()

        epoch_images.append(grid)


plt.figure(figsize=(20, 5))
steps = [0, 4, 9, 14, 19]  
for i, idx in enumerate(steps):
    plt.subplot(1, len(steps), i + 1)
    plt.imshow(epoch_images[idx].permute(1, 2, 0).cpu())
    plt.axis("off")
    plt.title(f"Epoch {idx+1}")
plt.savefig(f"{save_dir}/comparison.png")
plt.show()

```