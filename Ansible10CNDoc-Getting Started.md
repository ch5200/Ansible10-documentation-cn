# </center>Ansible 10 中文文档</center>

## Ansible 入门教程

### 开始使用Ansible

        Ansible自动管理远端的系统，并控制其所需预期状态。

![Basic components of an Ansible environment include a control node, an inventory of managed nodes, and a module copied to each managed node.](https://docs.ansible.com/ansible/latest/_images/ansible_inv_start.svg)

        如上图所示，大部分的Ansible环境有三个主要的组件。

1. ##### 控制节点
   
   控制节点是指：安装了Ansible的系统节点。你将在该节点上运行Ansible指令如`ansible`或者`ansible-inventory`

2. ##### 清单(Inventory)
   
   清单是指：逻辑组织的被管理的远程节点的列表。你可以在管理节点上创建远程主机的Ansible部署清单。
   
   注：Inventory更直观的翻译为库存，在此借用库存的概念，主要用于描述某个物体的所拥有的特性集合，如属性集合等。

3. ##### 受管节点
   
   受管节点是指：被Ansible管理的远程系统或者主机。

#### Ansible简介

        Ansible提供开源的自动化部署方案，它可以减低部署的复杂度以及可以在大部分安装了python的环境运行。使用Ansible几乎可以自动化任何的任务。如下是Ansible的常见任务：

- 消除重复性操作，并简化工作流

- 管理及维护系统配置

- 持续性的部署复杂的应用软件

- 执行零停机滚动更新

        Ansible使用简单的人类可读的，称为“playbooks”的脚本去自动你的任务。你在playbooks中定义本地系统或者远程系统的期望状态，Ansible确保系统保持在该状态。

作为一个自动化技术，Ansible以如下的原则进行设计：

##### 无代理的技术架构

        避免在IT基础架构中安装额外的软件，减低维护成本。

##### 简单化

        自动化的playbooks使用简单明了的YAML语法来编码，就如读取普通文档一样。并且Ansible也是去中心化的，使用现有的当前SSH的凭据来访问远程机器。

##### 可扩展的并具备灵活性

        通过支持大范围的操作系统，云平台及网络设备的模块式设计，可以快速，容易的自动扩展系统。

##### 幂等性及可预测性

        当你的系统处于playbooks描述的预期状态时，Ansible不会进行任何的操作，甚至playbooks运行了多次。

#### 使用Ansible开始自动化

        如下将通过创建一个自动化项目，创建描述清单，并创建一个“hello World”的playbook来开始Ansible自动化。

1. 安装Ansible（需要python环境）
   
   ```
   pip install ansible
   ```

2. 在你的文件系统创建项目文件夹
   
   ```
   mkdir ansible_quickstart && cd ansible_quickstart
   ```

        通过使用单文件夹架构，可以更容易的进行代码管理及重复使用、分享自动化内容。

#### 创建资源清单

        清单用于集中的以文件方式组织被管节点，向Ansible提供系统信息以及网络位置。通过清单文件，Ansible可以通过一个指令管理大量的主机。

        为了完成后继的步骤，你将需要至少一个主机系统的IP地址或者FQDN。基于演示的目的，该主机可能是在本地运行的一个容器，或者一个虚拟机。同时，你需要确保你的公钥SSH Key（通过Key方式登录）已经添加到每个被管主机`authorized_keys`文件中。

继续以下步骤创建清单：

1. 在你前些步骤创建的`ansible_quickstart`文件夹中创建一个名为`inventory.ini`的文件。

2. 添加新的`[myhosts]`配置组到`inventory.ini`文件，并指定每个被管主机的IP地址或者FQDN名称。
   
   ```
   [myhosts]
   192.0.2.50
   192.0.2.51
   192.0.2.52
   ```

3. 核实资源清单
   
   ```
   ansible-inventory -i inventory.ini --list
   ```

4. Ping你资源清单中的`myhosts`资源组
   
   ```
   ansible myhosts -m ping -i inventory.ini
   ```
   
   注意：
   
   如果你的管理节点和被管节点的用户名是不一样的，那么需要在`ansible`指令中附加`-u`选项。
   
   ```
   192.0.2.50 | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": false,
       "ping": "pong"
   }
   192.0.2.51 | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": false,
       "ping": "pong"
   }
   192.0.2.52 | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": false,
       "ping": "pong"
   }
   ```

##### 资源清单文件的格式（INI或YAML）

        你可以使用`INI`格式或者`YAML`格式创建资源清单文件。在大部分的案例中，例如前面的步骤，在一个少数受管的节点群中使用`INI`文件格式会比较直接和易读。

        如果你的受管节点群非常大，或者持续增长中，那么创建`YAML`格式的资源清单文件将是个比较明智的选项。如下面是一个跟上面的`inventory.ini`文件功能一样的，但是`ansible_host`字段额外声明了被管节点的名称及用户名：

```
myhosts:
  hosts:
    my_host_01:
      ansible_host: 192.0.2.50
    my_host_02:
      ansible_host: 192.0.2.51
    my_host_03:
      ansible_host: 192.0.2.52
```

##### 创建资源清单的小技巧

- 确保资源组名是“望文知义”的，以及是唯一的，并且资源组名是大小写敏感的。

- 避免使用带空格的、连字符的以及数字开头的组名。（如不要使用`19th_floor`而应该使用`floor_19`）

- 在你的资源清单中，分组的依据应该具备逻辑性。比如通过什么类型（What）、在那个位置（Where）、或者什么阶段（When）等来组织。
  
  **What**
  
  根据拓扑结构来组织，如：db,web,leaf,spine
  
  **Where**
  
  通过物理地址来组织，如：datacenter,region,floor,building
  
  **When**
  
  通过项目阶段来组织，如：development,test,staging,production

###### 使用元义组（metagroups）

通过如下的语法来创建metagroup，以便于组织你资源清单中的多个组：

```
metagroupname:
  children:
```

如：以下的资源清单描述一个数据中心的基础架构。这个清单例子包含一个`network`元义组，该组内包含了所有的网络设备；以及一个`datacenter`元义组，该组包含了所有的web服务器以及前面的`network`组。

```
leafs:
  hosts:
    leaf01:
      ansible_host: 192.0.2.100
    leaf02:
      ansible_host: 192.0.2.110

spines:
  hosts:
    spine01:
      ansible_host: 192.0.2.120
    spine02:
      ansible_host: 192.0.2.130

network:
  children:
    leafs:
    spines:

webservers:
  hosts:
    webserver01:
      ansible_host: 192.0.2.140
    webserver02:
      ansible_host: 192.0.2.150

datacenter:
  children:
    network:
    webservers:
```

###### 声明变量

变量用于设置（声明）被管节点的某个属性值，比如IP地址，FQDN名，操作系统名称，SSH用户名等。这样你在运行Ansible指令的时候不需要传输这些参数。（从console到process）

变量可以附加到特定的主机：

```
webservers:
  hosts:
    webserver01:
      ansible_host: 192.0.2.140
      http_port: 80
    webserver02:
      ansible_host: 192.0.2.150
      http_port: 443
```

变量也可以附加到一个组的所有主机上

```
webservers:
  hosts:
    webserver01:
      ansible_host: 192.0.2.140
      http_port: 80
    webserver02:
      ansible_host: 192.0.2.150
      http_port: 443
  vars:
    ansible_user: my_server_user
```

#### 创建playbook

Playbooks使用`YAML`格式，是Ansible自动化管理及部署被管节点的蓝图。

**Playbook**

一个定义了为取得整体目标的，Ansible将会自顶向下，顺序实施的操作，的剧本。

**Play**

一个映射到资源清单中被管节点的任务的顺序列表

**Task**

一个定义为单独模块的系列Ansible将要实施的操作的引用

**Module**

模块是指在被管节点上Ansible将运行的代码或二进制文件单元

Ansible的模块是以FQCN（ Fully Qualified Collection Name）即完全限定集名称来进行分组和组织。

完成以下的步骤来创建playbook，这个playbook将会pings你的主机，并且打印一个“Hello world”信息：

1. 在你的`ansible_quickstart`文件夹中创建一个名为`playbook.yaml`的文件，并输入以下内容：
   
   ```
   - name: My first play
     hosts: myhosts
     tasks:
      - name: Ping my hosts
        ansible.builtin.ping:
   
      - name: Print message
        ansible.builtin.debug:
         msg: Hello world
   ```

2. 运行你的playbook
   
   ```
   ansible-playbook -i inventory.ini playbook.yaml
   ```
   
   Ansible将会返回如下的输出：
   
   ```
   PLAY [My first play] ****************************************************************************
   
   TASK [Gathering Facts] **************************************************************************
   ok: [192.0.2.50]
   ok: [192.0.2.51]
   ok: [192.0.2.52]
   
   TASK [Ping my hosts] ****************************************************************************
   ok: [192.0.2.50]
   ok: [192.0.2.51]
   ok: [192.0.2.52]
   
   TASK [Print message] ****************************************************************************
   ok: [192.0.2.50] => {
       "msg": "Hello world"
   }
   ok: [192.0.2.51] => {
       "msg": "Hello world"
   }
   ok: [192.0.2.52] => {
       "msg": "Hello world"
   }
   
   PLAY RECAP **************************************************************************************
   192.0.2.50: ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
   192.0.2.51: ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
   192.0.2.52: ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
   ```
   
   在上面的输出，你可以看到：
   
   - 你给每个剧本、任务的名称。你应该使用描述性的望文知义的名称，这样可以便于检查或者playbooks排错
   
   - 隐式定义及运行的“Gathering Facts”任务。默认Ansible会收集你的资源清单中定义的信息，以便用于playbook
   
   - 每个任务的状态。每个任务的状态都为`ok`时说明该play成功运行。
   
   - Play Recap（执行回顾）总结每个被管主机上运行的playbook定义的所有任务的结果。在此例子中，这里有三个任务，所以`ok=3`表明每个任务都成功执行。
   
   恭喜你，到此你成功完成开始使用Ansible的任务。

#### Ansible基础概念

以下是Ansible中常见的基本概念。在使用Ansible或者阅读文档前，你应该理解这些基本概念的含义。

- [Control node（控制节点）](#control_node)

- [Managed nodes（被控节点）](#managed_nodes)

- [Inventory（资源清单）](#inventory)

- [Playbooks（操作剧本）](#playbooks)
  
  - [Plays（操作演义）](#plays)
    
    - [Roles（角色）](#roles)
    
    - [Tasks（任务）](#tasks)
    
    - [Handlers（处理器）](#handlers)

- [Modules（模块）](#modules)

- [Plugins（插件）](#plugins)

- Collections（采集集合）

 

**Control node（控制节点）** 

<div><a name="control_node"></a></div>
控制节点是指你运行Ansible 命令行工具（CLI tools）的（`ansible-playbook`，`ansible`，`ansible-vault`等）机器。你可以使用任何满足软件需求并可运行Ansible的计算机作为控制节点。你还可以在容器中运行Ansible，通常也被称作Execution Enviroments（可执行环境）。

多控制节点是被允许的，但是Ansible自身并不协调多节点。

**Managend nodes（被管节点）**

<div>
<a name="managend_nodes"></a>
</div>
被管节点通常也被引用为hosts（主机），它们是你用Ansible管理所指向的目标设备（如服务器、网络设备、或者任意的计算机）

通常情况下，Ansible并不在被控节点上安装。直到你使用`ansible-pull`指令推送，但这是罕见的配置，并非建议步骤

**Inventory（资源清单）**

<div>
<a name="inventory"></a>
</div>

资源清单是指：一个由单独的或多个资源清单提供的被管节点列表。你的资源清单可以指定特定于每个节点的信息，如IP地址；也可以指定特定信息给分组。两种方式都可以在剧本演义及批量变量设定任务中进行节点筛选。

**Playbooks（操作剧本）**

<div>
<a name="playbooks"></a>
</div>

操作剧本（playbook）包含操作演义（play）。操作演义是Ansible执行操作的基本单元。这是一个执行的概念，也是一个描述`ansible-playbook`如果操作的文件。

PLaybooks使用YAML格式，这使得文件可以便于阅读、理解及分享。

**Plays（操作演义）**

<div>
<a name="plays"></a>
</div>

操作演义（Play）是Ansible的主要执行内容，这个playbook的对象映射被管节点（主机）到任务。一个操作演义包含变量、角色、任务的顺序执行列表，并且可以（允许）重复执行。它基本上由所映射的主机及任务上的隐式循环组成，并定义如何迭代它们。

**Roles（角色）**

<div>
<a name="roles"></a>
</div>

角色是指一个可重复在操作演义（play）内部使用的Ansible内容（如tasks，handles，variables，plugins，templates以及files）的限量分发。

为了使用角色资源，角色其本身需要导入到演义中。

**Tasks（任务）**

<div>
<a name="tasks"></a>
</div>

任务是一个应用到被控节点的预定义好的操作。你可以通过使用`ansible`或者`ansible-console`及临时指令执行一次性的单独的任务。（两个都创建虚拟的演义）

**Handlers（处理器）**

<div>
<a name="handlers"></a>
</div>

任务的一种特殊形式，仅在前一个任务的状态通知为“产生变更状态时”执行。（也就是前一个任务状态变更时将会触发执行）

**Modules（模块）**

<div>
<a name="modules"></a>
</div>

模块是指每个任务中定义的在每个被管节点上的相关完成动作，这些动作具体是指由Ansible复制的（如果需要）并在每个被控节点上执行的代码或二进制程序。

每个模块都有其特殊的应用：从在一个特定类型的数据库上管理用户到在特定的网络设备上管理VLAN接口等等。

你可以以一个任务的方式触发单独的模块，或者在playbook中触发多个不同类型的模块。Ansible模块根据集合来进行分组。要想知道Ansible有多少种集合，请参看[集合索引](https://docs.ansible.com/ansible/latest/collections/index.html#list-of-collections)。

**Plugins（插件）**

<div>
<a name="plugins"></a>
</div>

扩展Ansible核心功能的代码片段。不同插件有不同的功能，比如链接插件可以控制你以何种形式链接到被管设备。使用过滤插件可以处理数据，甚至控制如何在控制台中显示内容（回调插件）。

**Collections（集合）**

<div>
<a name="collections"></a>
</div>

一种分发的Ansible内容格式，可以包含playbooks，roles，modules及plugins。你可以通过Ansible Galaxy安装并使用集合。
