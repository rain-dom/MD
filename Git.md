## 结构



每一个版本存储之前版本的索引

![git](E:\deng\deng\image\git.png)

![git](E:\deng\image\git.png)

## 常用命令

git --version 	康康版本

echo 'dzp first input' > README.MF

### 创建与提交

|                                        |                    |
| -------------------------------------- | ------------------ |
| git init dzp                           | 创建仓库           |
| git config --global user.name "chelly" | 设置全局名称       |
| echo 'dzp first input' > README.MF     | 写入文件           |
| git add -A                             | 全部加到暂存区     |
| git reset                              | 重置暂存区         |
| git rm --cached README.MF              | 删除暂存           |
| git commit -m "注释"                   | 提交到本地库并注释 |
| git log（空格下一页。b上一页。q退出）  | 日志（由近到远）   |

### 日志展示

|                          |              |
| ------------------------ | ------------ |
| git log --pretty=oneline | 一行展示日志 |
| git --oneline            |              |
| git reflog               | 展示head     |

### 切换版本

|                         |              |
| ----------------------- | ------------ |
| git reflog              | 展示head     |
| git reset  --hard  索引 | 同步更新     |
| git reset  --mixed      | 工作区不同步 |
| git reset  --soft       | 只变本地库   |
| git reset  --HEAD       | 恢复暂存区   |

|                          |                    |
| ------------------------ | ------------------ |
| git diff test2.txt       | 比较工作区与暂存区 |
| git diff                 | 比较全局           |
| git diff 历史版本 文件名 | 比较暂存区与工作区 |

### 分支

|                            |              |
| -------------------------- | ------------ |
| git branch -v              | 查看所有分支 |
| git branch 分支名          | 创建分支     |
| git checkout 分支名        | 切换当前分支 |
| cat a.txt                  | 看文件记录   |
| 冲突时                     |              |
| git merge branch_1         | 在主线中执行 |
| git commit -m "已解决冲突" | 不带文件名   |

### 远程库

|                             |                      |
| --------------------------- | -------------------- |
| git remote -v               | 查看别名             |
| git remote add 别名 地址    | 起别名               |
| git push 远程别名 推送分支  | push                 |
| git clone 地址              | fetch                |
|                             |                      |
| git fetch 远程别名 推送分支 | pull(不同步到工作区) |
| git checkout test/master    | 确认内容             |
| git merge test/master       | 合并                 |
| git pull 远程别名 推送分支  | 拉取远程库的内容     |

### 克隆

1.初始化本地库

2.内容拉取并起别名



### 团队协作

管理你的凭据

setting -> manage access -> 复制邀请链接

#### 冲突

B提交了更改之后，A对同一文件的修改提交会失败

需要先拉取下来解决冲突

再提交（不加文件名）



#### 跨团队协作

团队B fork过来

再pull过去

#### 免密操作

cd ~ 进入主目录

ssh-keygen -t rsa -C 591878545@qq.com

C:\Users\win\.ssh

复制到setting中的SSH



git remote add origin_ssh git@github.com:rain-dom/git.git

### IDEA集成

vertion control 中设置

#### **本地库初始化**

VCS -> import into version control -> create git repository

#### 拉取远程库

在本地库中打开Git Bash

git pull git@github.com:rain-dom/git.git --allow-unrelated-histories

#### 推送本地库同步

git push -u ssh master -f

开发中一般先pull,再push,防止冲突导致传不上去