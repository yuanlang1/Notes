# $git$学习  

**$git$学习过程中的一些重要的命令**  

## 配置$git$  

设置你的名称和邮箱地址

```git
在git Bash 中输入：
git config --global user.name "name"
git config --global user.email "email"

```

---

## 创建版本库

什么是版本库呢？版本库又名仓库，英文名repository，你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。  

创立的版本库会放在Desktop中，即桌面上

创建版本库的过程为：

```git
创建一个加test的版本库：

mkdir test
cd test
pwd
git init

```

**第一步**：
先找一个合适的地方，创建一个空目录

```git
请确保目录名（包括父目录）不包含中文。
mkdir 目录名
cd 目录名
pwd

例如：
mkdir a
cd a
pwd

```  

**第二步**：
把这个目录变成Git可以管理的仓库
`
git init 创建好空的仓库，并生成.git目录，用于git来跟踪管理版本库，可通过ls -ah查看
`

---

## 添加文件到git仓库中

``` git
git add test.txt
git add learn.txt
git commit -m "add test.txt learn.txt"
```

创建一个文件，将文件添加到仓库中即可  

**第一步**：
把文件添加到仓库中：

``` git
git add <file>
git add test.txt 将建好的test.txt通过git add提交到仓库中
```

**第二步**：
把文件提交到仓库中：

```git
git commit -m <message>
例如：
git commit -m "add test.txt"
```  

结果如下：
![结果](images/2023-07-21-17-58-59.png)

commit可以一次提交很多文件，所以你可以多次add不同的文件  

---

## 时光机穿梭  

查看工作区的状态，查看修改内容  

**查看是否修改**：

```git
git status 让我们时刻掌握仓库当前的状态
```

比如说我将test.txt中的内容修改，但是没有提交：
![结果](images/2023-07-21-19-19-18.png)  

**查看修改的内容**：

```git
git diff 查看修改的内容
git diff test.txt  查看test.txt 修改了什么内容
```

---

## 提交修改  

提交修改和提交新文件一样  

```git
git add test.txt 添加
git status 查看当前仓库的状态
git commit -m "add" 提交到库中
git status 查看当前仓库的状态
```

---

## 版本回退  

```git
git log 查看之前的记录
git reflog 查看未来的记录
```

**查看历史记录**：
`git log 查看历史记录`  
![查看历史记录](images/2023-07-21-19-36-52.png)  

**修改输出内容**：
`git log --pretty=oneline 只输出commit id`
![只输出一行](images/2023-07-21-19-39-33.png)

**回退**：

```git  
git reset --hard HEAD^ 回退到上一个版本 
git reset --hard <commit id> 看commit id的前几位
回退到上上个版本就为HEAD^^,回退到上100个版本就为HEAD~100

再使用cat查看文件中的内容
cat test.txt

git reflog 查看未来的版本
git reset --hard <commit id> 看未来的commit id的前几位
```  

---

## 工作区和缓冲区  

**工作区**：
我的test文件夹就是一个工作区：
![工作区](images/2023-07-21-20-22-32.png)  

**缓冲区**:

```git
用git add把文件添加进去，实际上就是把文件修改添加到暂存区(stage/index)
用git commit提交更改，实际上就是把暂存区的所有内容提交到当前分支
```

![缓冲区](images/2023-07-21-20-23-59.png)  

---

## 撤销修改

```git
当readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态
git checkout -- test.txt 丢弃工作区的修改

readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态
git reset HEAD test.txt  先把暂存区的修改回退到工作区
git checkout -- test.txt 丢弃工作区的修改
```  

**小结**:

```git
场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD <file>，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。
```  

---

## 删除文件  

**文件管理器中的删除**：

```git
添加test2.txt
cp test.txt test2.txt
git add test2.txt
git commit -m "add test2/txt"

文件管理器中的删除
rm test2.txt

```

在文件管理器中删除后，使用`git status`命令会告诉你那些文件被删除了
![文件删除](images/2023-07-21-20-45-19.png)  

**两种选择**：
一是确实要从版本库中删除该文件，那就用命令`git rm`删掉，并且`git commit`，将文件从版本库中删除：

```git
git rm test2.txt 告诉缓冲区
git commit -m "delete test2.txt" 将test2.txt从版本库中删除
```

另一种情况是删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本：

```git
git checkout --test2.txt 版本库中的还原回来
```

---

## 远程仓库  

使用`github`当做远程仓库  

**创建SSH Key**，在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有id_rsa和id_rsa.pub这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：
`ssh -keygen -t rsa -C "2392239422@qq.com"`  

**Add SSH Key**，在github上添加``.ssh`中的`id_rsa.pub`的文件内容  
![添加ssh key](images/2023-07-21-20-59-21.png)  

**为什么GitHub需要SSH Key呢？因为GitHub需要识别出你推送的提交确实是你推送的，而不是别人冒充的，而Git支持SSH协议，所以，GitHub只要知道了你的公钥，就可以确认只有你自己才能推送。**  

**添加远程仓库**：
已经建立好远程仓库  
![远程仓库](images/2023-07-21-21-08-50.png)

可以从这个仓库克隆出新的仓库，也可以把一个已有的本地仓库与之关联，然后，把本地仓库的内容推送到GitHub仓库。

**关联本地仓库**：

```git
git remote add origin git@github.com:yuanlang1/test.git 关联本地库
git push -u origin master 把本地库的所有内容推送到远程库master分支上
```  

![推送](images/2023-07-21-21-14-36.png)  
