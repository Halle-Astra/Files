# SVN(SubVersion)的使用
SVN(SubVersion)是一种版本管理工具，即类似Git的一种存在，其各命令均与git相似，Git适用于多人开发，方便快捷，而SVN适用于对权限管理较高要求以及企业内部使用。

如果是简单使用，可直接阅读[基础使用](basic_usage)部分。
## SVN命令行工具的安装
### Win下安装SVN命令行工具
1. 命令行工具的下载链接：https://sourceforge.net/projects/win32svn/
2. 将安装目录下的bin目录添加到环境变量path中
### Linux下安装SVN命令行工具
有些Linux版本自带SVN，此时可以输入`svn --version`来检查SVN并查看版本。
1. `yum install subversion`或`apt-get install subversion`可进行安装
## SVN可视化工具的安装
1. 下载https://osdn.net/projects/tortoisesvn/storage/
2. 其使用参考https://www.runoob.com/svn/tortoisesvn-intro.html；下文将仅介绍命令行工具的使用。

## SVN命令行工具的使用
### 创建版本库【服务器管理员】
1. `svnadmin create ./code_svn`:在服务器上用SVN创建一个空文件夹【版本库】用于存放代码
2. 配置`./code_svn/conf/authz`后版本库即可给各用户使用

### <span id = 'basic_usage'>SVN的基础使用</span>
这里假定服务器管理员已经配置好一个版本库`xxx`【类似git服务器上已经有了个repository】，以下我们先`mkdir code_svn`，`cd code_svn`，以`code_svn`文件夹为例子。

SVN一般是以文件夹的形式作为版本管理的物理存在形式。故而建议在服务器中`xxx`有个类似`v1`的文件夹作为真正存放代码的地方，而非将代码直接放置在`xxx`下。【为了方便之后分支的建立】

1. `svn checkout https://.../svn/xxx`：开始修改代码前，从版本库`xxx`【这个东西要找服务器管理员要，且需要个账号与密码】拷贝代码下来【类似于`git clone`】，得到的称为“工作副本”。执行这个句子后，可能会先要验证电脑账户名，电脑密码，svn账户名，svn密码。仅需要一次。
2. `svn add filename_or_folder`：将`filename_or_folder`添加到版本控制中
3. `svn status`：查看文件状态，若状态为`?`则需要用`svn add`将文件添加至版本控制中；若状态为`A`，则表示已添加至版本控制中。
4. `svn commit -m "comments"`：修改代码后，提交文件修改，类似【`git commit -m`】， `"comments"`内容是本次提交的备注说明
5. `svn revert filename`：撤销对`filename`的所有修改，若撤销对某个文件夹下所有文件的修改，可使用`svn revert -R foldername`.
6. `svn update`：对工作副本进行更新，与版本库同步。需要先`cd xxx`进入工作副本文件夹

### SVN提交时发生冲突
当一个用户`commit`后版本库的版本默认会自动+1【从0起算】.

假设A，B两人均已`checkout`版本13的代码，且两人均对代码进行了修改，A先提交了修改(commit)，然后B再进行提交，此时，由于A提交了，A提交后服务器上的版本为14，而B提交的依然是对13进行的修改，此时B提交会遭到SVN的拒绝，即发生了冲突。此时有如下解决方案
#### 方案1
1. `svn revert filename`：放弃对`filename`的所有修改，称为版本13
2. `svn update`：然后通过`update`使其成为版本14
3. 然后在版本为14的`filename`上进行修改后提交

#### 方案2
1. commit失败后，B用户直接`svn update`，此时会生成如下文件`filename.mine`--13版本内容+B用户的修改；`filename.r13`--13版本内容；`filename.r14`--14版本内容；`filename`--14版本内容与B用户的修改以某种方式合并后的结果，结果示例如下：
<pre >
<<<<<<< .mine
hello svn======
hhhhhhh>>>>>>> .r14
</pre>

这里，`<<<<<<<.mine   >>>>>>>.r14`包着的是合并的内容，`======`是分隔符。

2. 与队友商量后，修改`filename`文件，然后执行`svn resolved`【或`svn resolve --accept working`】，告知SVN冲突已解决，此时除了`filename`外的三个文件将消失。
3. `svn commit -m "comments"`提交修改。

### 分支的建立与合并
1. `cd xxx`;`mkdir branches`;`cd branches`;`mkdir branch_learn_1`;`cd ..`;`svn add branches`;`svn commit -m "mkdir branches and branch_learn_1"`：创建需要的文件夹并将这些修改添加到版本库中
2. `svn copy v1 branches/branch_1`：将创建分支`branch_1`，并把`v1`的内容拷贝至`branch_1`
2. `svn commit -m "Add branch_learn_1"`：创建分支后需要提交到版本库
3. 进行修改和提交
4. `svn update`：更新当前版本
5. `cd ....../xxx/v1`
6. `svn merge ../branches/branch_learn_1`：将分支合并到`v1`
7. `svn commit -m "Merge branche_learn_1"`：合并后提交到版本库

### 标签或Release的创建
对于一些重要的版本，我们可能需要将其单独标记或作为一个发布版本，在SVN中并没有专门的命令，而是直接采用文件拷贝的方式来实现打标签
1. `svn copy v1 tags/v1.0`
2. `svn commit -m "tags v1.0"`

### 恢复一个已提交的版本【版本回退,reverse merge】
`svn merge -r 22:21 filename`：文件`filename`将会从22版本回退至21

### 查看文件和文件夹信息
`svn log`,`svn diff`,`svn cat`,`svn list`均会在文件系统常见功能的基础上加上历史版本信息。详细内容参考https://www.runoob.com/svn/svn-show-history.html。
