## 第一节、Git本地

```git
 git init  :git的初始化
其他参数：
--initial-branch 初始化的分支，不想要默认的master
--bare 创建一个裸仓库（纯git目录，没有工作目录）
--template 可以通过模板创建预先建好的自定义git目录

```

```git
低配置会覆盖高配置
local-->global-->system
```

```git
改变的是config文件
git config 用户名配置
-- user.name "iceberg"
-- user.email iceberg@qq.com
```

![image-20220524115824945](E:\前端\git_10\image\image-20220524115824945.png)

```git
instead of 配置（ssh >>https）
git config -- url.git@github.com:.insteadOf https://github.com/
```

![image-20220524121810130](E:\前端\git_10\image\image-20220524121810130.png)

```git
git 命令别名:
git config -- alias.cin  "commit --amend  --no-edit"
就相当于
cin==commit --amend --no-edit
```

![image-20220524122146423](E:\前端\git_10\image\image-20220524122146423.png)

```git
git Remote：本地和远程一些关联信息
查看 /帮助 remote 
git remote -v /-h
添加 remote <命令> [name] [url]
git remote <add> origin_ssh git@github.com:git/git.git
git remote <add> origin_http https://github.com/git/git.git
同一个Origin设置不同的Push和Fetch URL
git remote add origin git@github.com:git/git
git remote set-url --add --push origin git@github.com:xiao/git.git
```

![image-20220524124611366](E:\前端\git_10\image\image-20220524124611366.png)

不安全，不推荐使用

```git
HTTP Remote
URL:https://github.com/git/git.git
免密配置：
内存：git config --global credential.helper 'cache --timeou=3600'
硬盘：git config --global credential.helper "store --fil  /path/to/credential-file"
不指定目录的情况默认是 ~/.git-credentials
将密钥信息指定文件中配置：
${scheme}://${password}@github.com
```

```ssh remote
SSH Remote
URL:git@github.com:git/git.git
免密配置
ssh可以通过公私钥的机制，将生成公钥存放在服务端，从而实现 免密访问。
dsa ,rsa,ecdsa,ed25519
默认使用rsa，现在优先ed25519
原因是由于一些安全问题，rsa有时候配置了公密钥，还是不能访问，是因为win改变了一些权限而导致这种问题的出现。
配置：
ssh-keygen -t ed25519 -C "your email@example.com" 密钥默认存放在~/.ssh/id_ed25519.pub
```

![image-20220524130524272](E:\前端\git_10\image\image-20220524130524272.png)

![image-20220524131021687](E:\前端\git_10\image\image-20220524131021687.png)



## 第二节、git命令原理

#### 四种objects组装,获取当前版本代码

```git
把文件添加进objects，也就是栈存区
git add <文件>
需要删除之前的操作
git rm -r --cached .
查看文件内容，虽然加密，但是还是可以通过id查看
git cat-file -p 文件名命令
```

![del](E:\前端\git_10\image\image-20220524133020266.png)

![成功](E:\前端\git_10\image\image-20220524133036603.png)

```git
然后可以提交到目录
git commit -m "add/update <文件名>"
Blob 存储文件的内容
Tree 存储文件的目录
Commit 存储提交信息，一个Commit可以对应唯一版本的代码
《由下往上找信息》
```

![image-20220524133507728](E:\前端\git_10\image\image-20220524133507728.png)

![image-20220524134109429](E:\前端\git_10\image\image-20220524134109429.png)

## 第三节、修改refs文件

```git
创建一个新分支：
git checkout -b <分支名称>
创建一个标签：
git tag <版本名称>
```

refs的内容对应的就是**Commit ID**,因此把ref当作指针，指向对应的Commit来表示当前Ref对应的版本。

不同种类的ref

refs/heads 前缀表示的是**分支**指向相同的commit，用于开发阶段，是可以不断添加Commit进行迭代

还有其他种类的ref，比如refs/tags 前缀表示的是**标签**，表示一个稳定版本，指向的commit是不会变更的,版本迭代。

![image-20220524135034242](E:\前端\git_10\image\image-20220524135034242.png)

#### Annotation Tag 

Annotation Tag 附注标签，可以提供一些额外的信息

```git
创建附注标签
git tag -a  
```

## 第四节、获取历史版本代码

**Commit里面存有parent commit字段通过commit的串联获取历史代码**

```git
因为内容变了，那么tree下的blob就会发生改变，不会影响之前的，就相当于copy从新记录了一遍。
```

```git
修改历史版本
1、commit --amend
通过这个命令可以修改最近一次的commit信息，修改之后commit id会发生变化
2、rebase
git rebase -i HEAD~3 可以实现对最近三个commit的修改
1）合并commit
2）修改具体commit message
3）删除某个commit
3、filter --branch
 删除所有提交中的某个文件或全局修改邮箱地址等操作
```

**新增的Object**
修改后Commit后我们发现git object又出现新的，原先的并没有被删除。所有引出**悬空的Object**,顾名思义就是没有ref指向的object

```git
git fsck --lost-found
```

**Git GC**

GC :删除一些不需要的object

Reflog :用于记录操作日志，泛指误操作后数据丢失，手动将日志设置为过期

```git
git reflog expire -expire=now --all
```

指定时间

git gc prune=now 指定修建多久之前的时间，默认是两周前

![image-20220524163737224](E:\前端\git_10\image\image-20220524163737224.png)



## 第五节、总结git原理图：

![image-20220524164132035](E:\前端\git_10\image\image-20220524164132035.png)

## 第六节、git 远程与多人协作

```git
Clone：拉取完整创库到本地目录。
Fetch: 将远端某些分支最新代码拉取到本地，不会执行merge操作
pull：拉取远端某分支，并和本地代码进行合并，操作等同于git fetch +git merge,或者可以git pull --rebase 完成git fetch+git rebase,这种可能会存在冲突
```

```git
push:将本地同步远程
git push origin masters
```

## 第7节、常见问题

1、为什么我配置了git,但是还是无法拉取代码？

- 免密认证没有配
- instead of 没有配置，配置是ssh，但是用的是HTTP协议

2、为什么我Fetch了远端分支，但是我看到本地当前的分支历史还是没有变化？

- fetch 会把代码拉取到本地的远端分支，但是并不会合并到当前分支，所以当前分支无变化，需要自己自行rebase操作



#### 团队合作

![image-20220524165746705](E:\前端\git_10\image\image-20220524165746705.png)

#### 集中式

![image-20220524165838982](E:\前端\git_10\image\image-20220524165838982.png)

#### 分支

![image-20220524170104926](E:\前端\git_10\image\image-20220524170104926.png)

#### 第8节、本地代码push到远程

**1、创建一个master分支**

git add <文件名>

git commit -m “add <文件名>”

git push origin master

![image-20220524172110234](E:\前端\git_10\image\image-20220524172110234.png)

**2、“更新”创建一个feature**

git checkout -b feature

git add <文件名>

git commit -m  "update <文件名> "

git push origin feature

![image-20220524172643134](E:\前端\git_10\image\image-20220524172643134.png)

## 第九节、代码合并

#### Fast-forward

```
git checkout -b <分支名>：创建一个分支
git add <文件> :添加文件进入obstal
git commit -m "分支名字" ：提交
git checkout <分支名字> ：然后切换分支
git merge <分支名字>  --ff-only :合并分支
```

![image-20220526085403554](E:\前端\git_10\image\image-20220526085403554.png)

![image-20220526085712637](E:\前端\git_10\image\image-20220526085712637.png)

#### Three-Way Merge

![image-20220526090547581](E:\前端\git_10\image\image-20220526090547581.png)

```git
git merge <节点名称> --no-ff :把节点名称与当前节点合并为一个
```

