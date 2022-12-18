#### git_basic
工作区 -> 暂存区 -> 版本库

#### git init :将当前目录配置成git仓库，信息记录在隐藏的.git文件中

#### 通过 git add 命令将文件加载到暂存区
如： git add readme.txt
#### 通过 git commit 命令将文件从暂存区添加到当前分支
如： git commit -m "给自己的提示信息备注"

#### 若此时将 readme.txt文件进行了修改，使用git status命令查看它的状态，
#### 再次将 readme.txt进行提交到暂存区
git add readme.txt
#### 通过 git diff XX 命令查看XX文件相对于暂存区修改了哪些内容
git diff readme.txt

#### git restore --stage 将文件从暂存区撤出，但不会修改文件，但是我还管理这个文件
#### git rm --cached XX 将文件从暂存区中撤出，不管理这个文件了

#### git add. 将所有文件都加入到暂存区
#### git log 查看当前分支的所有版本，从下往上看

#### git log --pretty=oneline 放在一行显示

#### git reset --hard HEAD ^ 或 git reset --hard HEAD~ 将文件回滚到上一个版本

#### git reflog 显示 head的所有移动记录（包括被回滚的版本）

#### git reset --hard 版本号  回滚到某一特定版本（取md5值的前7位）

#### git restore 将工作区（尚未加入到暂存区）的修改内容撤回

#### rm a.txt b.txt 删除 两个文件，同时 git add  a.txt b.txt 存入到暂存区，不仅可以增加文件，也可以删除文件

#### git checkout branch_name :切换到branch_name这个分支

#### git branch :查看所有分支和当前所处分支

#### git branch branch_name : 创建新分支

#### git merge branch_name :将分支branch_name合并到当前分支

#### git branch -d branch_name ： 删除本地仓库的branch_name分支

#### git push -d origin branch_name : 删除远程仓库中的branch_name分支


#### SSH 方式连接云端仓库
#### 公钥在  .ssh/文件夹中， 通过 cat id_rsa.pub查看公钥
#### 将tmux中的文本复制到 windows中： 按住shift 鼠标拖住选中文本，按ctrl+ins复制文本， 回到windows中，按shift+ins粘贴文本

#### git remote add origin git@git.acwing.com:Jacky/git_project2.git 将本地文件夹与云端仓库建立连接
#### git push -u origin master 将本地文件夹push到云端仓库中（注意是master）

#### 从云端将文件夹clone到本地 git clone git@git.acwing.com:Jacky/git_project2.git
#### 当查看log时， 按 q 退出

