# Ansible 10 中文文档

## Ansible 安装指南

### 安装 Ansible

Ansible是一个无代理的自动化工具，可以安装在单个主机上

从控制节点，Ansible可以通过SSH、远程Powershell和许多其他的管理工具，远程管理整个机器群和其他设备(称为托管节点)，所有这些都来自一个简单的命令行界面，不需要数据库或守护进程。

- [管理节点要求](#control_node_req)
- [受管节点要求](#managed_node_req)
- [节点要求概述](#node_req_summary)
- [选择Ansible包和版本安装](#select_version_install)
- [使用pipx安装和升级Ansible](#pipx_install)
  - [安装Ansible](#pipx_inst_Ansible)
  
  - [升级Ansible](#pipx_up_Ansible)
  
  - [安装额外的Python依赖](#pipx_inst_python_ext)
  
- [使用pip安装和升级Ansible](#pip_inst_Ansible)
  - [定位Python](#locate_Python)
  
  - [确保`pip`可用](#check_pip)
  
  - [安装Ansible](#pip_inst_ansible)
  
  - [升级Ansible](#pip_up_Ansible)
- [安装Ansible到容器](#inst_to_container)
- [为开发安装](#ins_dev)
  - [使用`pip`从Github安装`devel`](#pip_ins_devel)
  
  - [从克隆运行`devel`分支](#run_from_clone)
- [确认你的安装](#confirm_ins)
- [增加Ansible命令shell补全功能](#add_argcomplete)
  - [安装`argcomplete`](#inst_argcomplete)
  
  - [配置`argcomplete`](#cfg_argcomplete)
    - [全局配置](#cfg_global)
    
    - [基于每命令配置](#cfg_cmd)
    
    - [通过zsh或tcsh使用`argcomplete`](#with_zsh`)

---

**管理节点要求**

<div><a name="control_node_req"></a></div>

对于你的控制节点(运行Ansible的机器)，你可以使用几乎任何安装了Python的类unix机器。这包括Red Hat、Debian、Ubuntu、macOS、bsd和Windows Subsystem for Linux (WSL)发行版下的Windows。没有WSL的Windows本身不支持作为控制节点;更多信息请看Matt Davis的博客文章。

**受管节点要求**

<div><a name="managed_node_req"></a></div>
受管节点不需要安装Ansible，但是需要Python去运行Ansible生成的Pyton代码。被管节点同时需要一个用户账号用于通过SSH远程连接到受管节点并具备交互POSIX Shell
此要求可能有特殊的例外，比如网络模块不需要python安装于被管理的设备

**节点要求概述**

<div><a name="node_req_summary"></a></div>

您可以在Ansible-core控制节点Python支持和Ansible-core支持矩阵部分中找到有关每个Ansible版本的控制和托管节点需求(包括Python版本)的详细信息

**选择Ansible包和版本安装**

<div><a name="select_version_install"></a></div>

Ansible的社区版本包以两种方式分发：
- `ansible-core`：包含最小的运行时包及内置功能的极简语言
- `ansible`：一个较大的包，涵括为自动化各种各样的设备增加了一个社区策划的Ansible集合的选择

可以根据自身合理的需求选择使用的版本，如下使用`ansible`作为包名，你也可以使用`ansible-core`替代，如果你需要使用最小的包来起步，并分开安装Ansible 集合 

`ansible`和`ansible-core`包可能包含在你系统的包管理器中，你可以通过首选的方式安装Ansible包，此文档仅包含官方支持的通过`pip`方式来安装python包

**使用pipx安装和升级Ansible**

<div><a name="pipx_install"></a></div>

在某些系统可能无法通过`pip`安装Ansible，那么可以使用`pipx`来代替，它是一个更广泛使用的安全带隔离的Python包安装工具。

本指南不包含如何安装`pipx`，如果需要请参看pipx安装指南。

**安装Ansible**

<div><a name="pipx_inst_Ansible"></a></div>

在你的环境中使用`pipx`安装完整Ansible包：

```python
$ pipx install --include-deps ansible
```

你也可以安装最小集`ansible-core`包：

```python
$ pipx install ansible-core
```

你也可以安装一个特定的`ansible-core`版本：

```python
$ pipx install ansible-core==2.12.3
```



**升级Ansible**

<div><a name="pipx_up_Ansible"></a></div>

升级一个现存的Ansible安装版本为最新的发行版本：

```python
$ pipx upgrade --include-injected ansible
```



**安装额外的Python依赖**

<div><a name="pipx_inst_python_ext"></a></div>

以下的例子为安装python额外的依赖包，比如安装`argcomplete`python包：

```
$ pipx inject ansible argcomplete
```

使用带`--include-apps`的选项让额外python扩展包的应用在你的PATH可用，这使得你可以从shell运行那些额外的应用。

```
$ pipx inject --include-apps ansible argcomplete
```



**使用pip安装和升级Ansible**

<div><a name="pip_inst_Ansible"></a></div>

**定位Python**

<div><a name="locate_Python"></a></div>

定位并记住你计划让Ansible使用的Python解析器的路径，如下面的例子使用`python3`，假设该解析器位于`/usr/bin/python3.9`那么你可以直接使用完整的路径来替代使用`python3`

**确保`pip`可用**

<div><a name="check_pip"></a></div>

检查并确认`pip`已经安装于你所引用的首选Python中：

```
$ python3 -m pip -V
```

如果一切正常，那么你将看到如下的信息

```
$ python3 -m pip -V
pip 21.0.1 from /usr/lib/python3.9/site-packages/pip (python 3.9)
```

如果看到如上类似信息，那么你可以继续下一步，如果你看到有类似`No module named pip`的报错，那么你需要先安装`pip`。这表示你需要额外的安装附加的OS包基于你首选使用的python版本。比如`python3-pip`或者直接从Python权威包站点安装最新版本的`pip`：

```
$ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
$ python3 get-pip.py --user
```

你可能需要额外的配置，请参看Python文档中的installing to the user site

**安装Ansible**

<div><a name="pip_inst_ansible"></a></div>

使用`pip`为你当前用户安装Ansible的全量包：

```
$ python3 -m pip install --user ansible
```

你可以安装`ansible-core`最小版本：

```
$ python3 -m pip install --user ansible-core
```

你也可以指定某个特定的版本：

```
$ python3 -m pip install --user ansible-core==2.12.3
```



**升级Ansible**

<div><a name="pip_up_Ansible"></a></div>

为了升级一个现存的安装版本，需要使用`--upgrade`参数：

```
$ python3 -m pip install --upgrade --user ansible
```



**安装Ansible到容器**

<div><a name="inst_to_container"></a></div>

你可以简单的创建一个可执行容器镜像或者直接使用现存的社区镜像作为你的控制节点，以替代直接手工安装到你的环境。请参看开始使用可执行环境的指南。

**为开发安装**

<div><a name="ins_dev"></a></div>

如果你准备测试新功能，修正bugs或其他的目的跟开发团队去维护核心代码，那么你可以从Github安装并运行源码

**使用`pip`从Github安装`devel`**

<div><a name="pip_ins_devel"></a></div>

你可以通过`pip`直接从Github安装`ansible-core`的`devel`版本分支

```
$ python3 -m pip install --user https://github.com/ansible/ansible/archive/devel.tar.gz
```

你可以修改上面命令行中的`devel`为任何的Github上存在的分支名称。

**从克隆运行`devel`分支**

<div><a name="run_from_clone"></a></div>

`ansible-core`是非常容易的从代码运行。你不需要`root`权限，并且不需要其他的软件，其他的守护进程或者数据库。

1. 克隆`ansible-core`仓库

   ```
   $ git clone https://github.com/ansible/ansible.git
   $ cd ./ansible
   ```

2. 设置Ansible运行环境

   - 使用Bash

     ```
     $ source ./hacking/env-setup
     ```

   - 使用Fish

     ```
     $ source ./hacking/env-setup.fish
     ```

   - 为了抑制假的警报和错误，使用`-q`参数

     ```
     $ source ./hacking/env-setup -q
     ```
   
3. 安装Python依赖
   
   ```
   $ python3 -m pip install --user -r ./requirements.txt
   ```
   
4. 更新本机的`ansible-core`的`devel`分支代码，使用pull-with-rebase所以任何的本地修改将被重播。

   ```
   $ git pull --rebase
   ```

   

**确认你的安装**

<div><a name="confirm_ins"></a></div>

你可以使用如下的检查版本的代码去测试Ansible是否安装正确

```
$ ansible --version
```

此命令显示的版本号为已安装的关联`ansible-core`包的版本号。如果要测试`ansible`包应该以如下的命令检查：

```
$ ansible-community --version
```



**增加Ansible命令shell补全功能**

<div><a name="add_argcomplete"></a></div>

你可以为Ansible命令行工具增加shell补全功能，通过安装一个可选的名为`argcomplete`的依赖。它兼容bash，并有限的支持zsh及tcsh

**安装`argcomplete`**

<div><a name="inst_argcomplete"></a></div>

如果你使用`pipx`安装指令：

```
$ pipx inject --include-apps ansible argcomplete
```

如果你使用`pip`安装指令：

```
$ python3 -m pip install --user argcomplete
```



**配置`argcomplete`**

<div><a name="cfg_argcomplete"></a></div>

有两个方式配置`argcomplete`:全局配置方式或者每命令行方式。	

**全局配置**

<div><a name="cfg_global"></a></div>

全局模式需要bash4.2以上

```
$ activate-global-python-argcomplete --user
```

上诉的命令将写入一个bash的补全文件到用户本地。使用`--dest`来变更位置，或者使用`sudo`来设置全局补全。

**基于每命令配置**

<div><a name="cfg_cmd"></a></div>

如果你没有bash4.2版本，那么你需要为每个命令注册每个独立脚本

```
$ eval $(register-python-argcomplete ansible)
$ eval $(register-python-argcomplete ansible-config)
$ eval $(register-python-argcomplete ansible-console)
$ eval $(register-python-argcomplete ansible-doc)
$ eval $(register-python-argcomplete ansible-galaxy)
$ eval $(register-python-argcomplete ansible-inventory)
$ eval $(register-python-argcomplete ansible-playbook)
$ eval $(register-python-argcomplete ansible-pull)
$ eval $(register-python-argcomplete ansible-vault)
```

你应该将上述的指令放到你的shell profile文件中如`~/.profile`或者`~/.bash_profile`

**通过zsh或tcsh使用`argcomplete`**

<div><a name="with_zsh"></a></div>

请参考argcomplete documentation
