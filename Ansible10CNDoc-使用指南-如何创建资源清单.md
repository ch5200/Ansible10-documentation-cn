# Ansible 10 中文文档

## Ansible使用指南

### 创建Ansible资源清单

#### 如何创建你的资源清单

通过使用一个称为资源清单的列表或者列表组，Ansible 可自动执行你基础设施中受管理节点或 "主机 "上的任务。你可以在命令行中传递主机名，但是大多数的用户会创建资源清单文件。你的资源清单定义你将要自动化的被管节点，使用主机组，你可以使用同时在多台主机上运行自动化任务。一旦你的资源清单定义完毕，你可以使用模式去选择要运行的主机和组。

最简单的资源清单是一个单独的文件包含主机列表及主机组。默认该文件的位置为`/etc/ansible/hosts`，你可以在命令行中通过`-i <path>`选项或者在配置中使用`inventory`

Ansible Inventory 插件支持多种格式和来源，使您的库存灵活可定制。随着清单的扩大，您可能需要不止一个文件来组织主机和组。下面是 /etc/ansible/hosts 文件之外的三个选项：

- 你可以创建一个目录并创建多个资源清单文件。参看在目录中组织资源清单。你可以使用不同的格式。（YAML，INI等）
- 你可以动态的拉取资源清单，例如，你可以使用动态清单插件来组织一个或多个云供应商。请参看使用动态资源清单。
- 你可以使用多个资源清单，包括动态清单及静态文件。参看传递多个资源清单。

---

**目录**

- [资源清单基础：格式，主机及组](#inv_basics)
  - [默认组](#def_group)
  - [主机在多个主机组](#host_in_mgroup)
  - [分组：父/子分组关系](#parent_child)
  - [添加主机范围](#host_range)

- [传输多个清单源](#pass_mul_source)
- [在目录中组织清单](#org_inv_indir)
  - [管理清单加载顺序](#mgr_load_order)
- [添加变量到清单](#add_var_to_inv)
- [分配变量给一台机器：主机变量](#host_var)
  - [清单别名](#inv_alias)
- [定义变量使用INI格式](#ini_in_var)
- [分配变量给多个主机：组变量](#var_group)
  - [继承变量值：为组中组分组变量](#gvar_in_group)
- [组织主机和主机组的变量](#org_hg_var)
- [如何合并变量](#var_merged)
  - [管理清单变量的加载顺序](#mgr_inv_var_order)
- [连接到主机：行为清单参数](#conn_2_host)
  - [非SSH连接类型](#non_ssh)
- [创建清单例子](#example)
  - [例子：一个环境一个资源清单](#example)
  - [例子：以功能分组](#g_by_fun)
  - [例子：以位置分组](#g_by_loc)

---

**资源清单基础：格式，主机及组**
<div><a name="inv_basics"></a></div>

你可以使用多个支持格式中的其中一个来创建资源清单文件。这个将根据你的清单插件来确定。常见的格式是INI和YAML格式。`/etc/ansible/hosts`可能组织如下：

```
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

括号中的标题是组名，用于对主机进行分类，并决定在什么时间出于什么目的控制什么主机。组名应遵循与创建有效变量名相同的准则。以下是相同功能的YAML格式

```
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
```



**默认组**

<div><a name="def_group"></a></div>

如果你没有在清单中定义任何的组，Ansible将会创建2个默认组：`all`和`ungrouped`。`all`组包含所有的主机，`ungrouped`组保存所有不包含于其他（除all外）任何组的主机。一台没有加入任何组的主机，同时属于`all`及`upgrouped`组。虽然 all 和 ungrouped 总是存在，但它们可以是隐式的，不会像 group_names 一样出现在组列表中。



**主机在多个不同的主机组**
<div><a name="host_in_mgroup"></a></div>

你可以将每个主机放入超过一个分组，也就是说一个主机可以属于多个分组，主机跟分组间是m：n的关系。比如：一台生产环境的web服务器在Atlanta的数据中心，那么可能会包含如下的组`[prod]`，`[atlanta]`，和`[webservers]`三个组。你可以依照如下的跟踪逻辑创建分组：

- 什么——一个应用程序，堆栈或微服务。（如数据库服务器，网页服务器等等）
- 哪里——一个数据中心或者一个区域，一个本地的DNS，存储等。（如东部、西部）
- 什么时候——开发阶段，为了避免在正式资源中出现测试。



扩展前面的YAML资源清单包含上述组：

```
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
east:
  hosts:
    foo.example.com:
    one.example.com:
    two.example.com:
west:
  hosts:
    bar.example.com:
    three.example.com:
prod:
  hosts:
    foo.example.com:
    one.example.com:
    two.example.com:
test:
  hosts:
    bar.example.com:
    three.example.com:
```



**分组：父/子分组关系**
<div><a name="parent_child"></a></div>

你可以在组之间创建父/子关系，父组同样称为嵌套组或者分组的组。例如：如果所有你的生产主机已经在`atlanta_prod`和`denver_prod`那么你可以创建一个`production`组，并且这个组包含上述两个组。这种方法可以减少维护工作，因为你可以通过编辑子组从父组中添加或删除主机。

为了创建父/子关系组：

- 使用INI格式，使用`:children`后缀
- 使用YAML格式，使用`children:`条目

如下是上述相同的资源清单，并且使用简单的父组`prod`和`test`组。这两个清单文件结果一样：

```
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
east:
  hosts:
    foo.example.com:
    one.example.com:
    two.example.com:
west:
  hosts:
    bar.example.com:
    three.example.com:
prod:
  children:
    east:
test:
  children:
    west:
```

使用子组，需要注意如下的属性：

- 任何一个主机属于子组，那么便自动成为父组的成员。
- 一个组可以有多个父组和子组，但两个组间不能称为循环关系。
- 主机也可以属于不同的组，但是运行时，主机仅能只有一个实例。Ansible将从多个所属分组中合并数据。



**添加主机范围**

<div><a name="host_range"></a></div>

如果你有很多主机，并带有相同的模式，你可以将它们作为一个范围添加，以替代单独分开列出。

使用INI：

```
[webservers]
www[01:50].example.com
```

使用YAML：

```
# ...
  webservers:
    hosts:
      www[01:50].example.com:
```

在定义主机的数字范围时，可以指定跨距（序列号之间的增量）：

使用INI：

```
[webservers]
www[01:50:2].example.com
```

使用YAML：

```
# ...
  webservers:
    hosts:
      www[01:50:2].example.com:
```

对于数字模式，可根据需要加入或删除前导零。范围是包含在内的。您还可以定义字母范围：

```
[databases]
db-[a:f].example.com
```



**传递多个清单源**

<div><a name="pass_mul_source"></a></div>

通过在命令行中提供多个清单参数，或通过配置 `ANSIBLE_INVENTORY` 参数，可以同时指向（汇入）多个清单源（清单插件支持的目录、动态清单脚本或文件）。当您希望指向（汇入）通常独立的分开的环境(如某个步骤环境和生产环境)同时执行特定操作时，这可能很有用。

在命令行中通过指定参数，针对两个不同的来源：

```
ansible-playbook get_logs.yml -i staging -i production
```



**使用文件夹组织清单**

<div><a name="org_inv_indir"></a></div>

你可以将多个清单合并到一个单独的目录中，最简单的实现是在一个目录中使用多个清单文件来替代一个清单文件。当仅有一个文件时，可能会因内容过大、过长而难于维护。如果你有多个团队或者多个自动化项目，那么每个团队使用一个清单文件或者每个项目一个清单文件，这样让大家更容易的找到关联的组或者主机。

你还可以在一个清单目录中将多种清单来源类型组合在一起，这将对组合动态和静态的主机，并将他们视为一个仓库非常有用。如下的资源清单目录组合一个清单插件源，一个动态的清单脚本，以及一个静态的主机文件：

```
inventory/
  openstack.yml          # configure inventory plugin to get hosts from OpenStack cloud
  dynamic-inventory.py   # add additional hosts with dynamic inventory script
  on-prem                # add static hosts and groups
  parent-groups          # add static hosts and groups
```

你可以通过以下的命令来指向（汇入）这个清单目录：

```
ansible-playbook example.yml -i inventory
```

你还可以在`ansible.cfg`文件配置你的资源清单目录，具体请参看[Configuring Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_configuration.html#intro-configuration)



**管理资源清单的加载顺序**
<div><a name="mgr_load_order"></a></div>

Ansible以文件名的ASCII字母的顺序加载资源清单文件。如果你在一个文件中或目录中定义了父组，并且在另外一个文件或者目录定义了子组，那么定义子组的文件或者目录必须先于父组所在文件或者目录先加载。如果父组所在文件先加载，将会出现如下错误：

```
Unable to parse /path/to/source_of_parent_groups as an inventory source
```

例如：如果你有一个名为`groups-of-groups`的文件，定义了一个`production`组，并且子组定义在一个名为`on-prem`的文件。那么Ansible将不能解析`production`组。为了避免这种情况，那么你可以通过在文件添加前缀的方式控制文件的加载顺序：

```
inventory/
  01-openstack.yml          # configure inventory plugin to get hosts from OpenStack cloud
  02-dynamic-inventory.py   # add additional hosts with dynamic inventory script
  03-on-prem                # add static hosts and groups
  04-groups-of-groups       # add parent groups
```



**添加变量到资源清单**
<div><a name="add_var_to_inv"></a></div>

你可以在资源清单中保存特定给某一个主机或者组的变量值，在开始前，你需要在资源清单中给主机或者组添加变量名。

我们给出了在主要的清单文件中添加变量的方式作为简要的例子，但是实际上在不同的主机和组变量文件中保存变量是一个更好的方式。将变量保存在主清单文件中仅仅是一个快速的方式。参看 [Organizing host and group variables](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#splitting-out-vars) 去获得如何使用`host_vars`目录来保存独立变量文件来存储变量值的方法。



**将变量附加到一个主机：主机变量**
<div><a name="host_var"></a></div>

你可以很简单的给某个主机指派变量，并且在后面的playbooks中使用。比如你可以在你的资源清单文件中通过如下的方式来指派：

INI格式：

```
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
```

YAML格式：

```1
atlanta:
  hosts:
    host1:
      http_port: 80
      maxRequestsPerChild: 808
    host2:
      http_port: 303
      maxRequestsPerChild: 909
```

像非标准SSH端口这样的唯一值可以很好地用作主机变量。你可以将它们添加到你的Ansible清单中，方法是在主机名后面加上端口号和冒号:

```
badwolf.example.com:5309
```

连接变量同样可以作为主机变量来使用：

```
[targets]

localhost              ansible_connection=local
other1.example.com     ansible_connection=ssh        ansible_user=myuser
other2.example.com     ansible_connection=ssh        ansible_user=myotheruser
```

注意：

如果在SSH配置文件中列出非标准SSH端口，`openssh`连接将找到并使用它们，但`paramiko`连接不会。

**清单别名**

<div><a name="inv_alias"></a></div>

你可以甚至在你的清单文件中通过主机变量的方式定义别名：

INI格式：

```
jumper ansible_port=5555 ansible_host=192.0.2.50
```

YAML格式：

```
# ...
  hosts:
    jumper:
      ansible_port: 5555
      ansible_host: 192.0.2.50
```

在上面的例子中，如果以主机名“jumper”来运行Ansible将会连接到192.0.2.50：5555



**用INI格式定义变量**

<div><a name="ini_in_var"></a></div>

在INI格式中，以`key=value`的语法格式来传递变量，根据不同的定义位置，有不同的解析。

- 当紧跟主机并在同一行上定义时，INI的变量作为Python的声明结构（如strings, numbers, tuples, lists, dicts, booleans, None）来解析。主机行接受多个`key=values`参数，为了作出区分，那么需要一个分隔符，如空格。但是有时候变量值中可能包含空格，那么需要使用引号（单引号或者双引号）来将带空格的内容值标明。
- 如果在`:vars`区域中定义，那么INI值将被认为时字符串。比如`var=FALSE`将创建一个值为FALSE的字符串，不同于主机行，在主机行中`:vars`区域仅允许一个条目，所以`=`后面的内容都认为是该条目的值。

如果INI清单中设置的变量值必须是明确确定的某种类型(例如，字符串或布尔值)，则始终在任务中使用过滤器指定该类型。在使用变量时，不要依赖INI清单中设置的类型。

为了防止变量类型的混淆，那么可以考虑使用YAML格式，YAML格式插件处理变量类型一致、正确。



**给多个主机指派变量：组变量**

<div><a name="var_group"></a></div>

如果在一个组中，所有的主机都共享一个变量值，那么你可以一次性的将该变量指派给整个组。

INI格式：

```
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
```

YAML格式：

```
atlanta:
  hosts:
    host1:
    host2:
  vars:
    ntp_server: ntp.atlanta.example.com
    proxy: proxy.atlanta.example.com
```

组变量是一次将变量应用于多个主机的方便方法。然而，在执行之前，Ansible总是将变量(包括清单变量)扁平化到主机级别。如果一个主机是多个组的成员，Ansible会从所有这些组中读取变量值。如果您在不同的组中为相同的变量分配不同的值，Ansible会根据合并的内部规则选择使用哪个值。



**继承变量：分组的分组的组变量**

<div><a name="gvar_in_group"></a></div>

你可以为父组指派变量（嵌套组或者分组的组）使用相同的`:vars`在INI中或者使用`vars:`在YAML格式中：

INI格式：

```
[atlanta]
host1
host2

[raleigh]
host2
host3

[southeast:children]
atlanta
raleigh

[southeast:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[usa:children]
southeast
northeast
southwest
northwest
```

YAML格式：

```
usa:
  children:
    southeast:
      children:
        atlanta:
          hosts:
            host1:
            host2:
        raleigh:
          hosts:
            host2:
            host3:
      vars:
        some_server: foo.southeast.example.com
        halon_system_timeout: 30
        self_destruct_countdown: 60
        escape_pods: 2
    northeast:
    northwest:
    southwest:
```

一个子组的变量将会比父组的变量有更高的优先级别（如果名称相同则覆盖掉父组变量）



**组织主机和组变量**
<div><a name="org_hg_var"></a></div>

尽管你可以通过主资源清单文件存储变量，但是存储分离的主机及组变量文件将让你可以更容易的组织你的变量值。你也可以在主机和组变量文件中使用lists及hash数据，这两个是无法在主清单文件中使用的。

主机及组变量文件，必须使用YAML格式，有效的扩展名为.yml，.yaml，.json或者没有扩展名。

Ansible加载主机和组变量文件，通过查找相对于清单文件或者playbook所在位置的相对路径。如果你的清单文件在`/etc/ansible/hosts`并有一个主机名为foosball,并且此主机属于两个组raleigh及webservers,那么主机将使用YAML文件的变量的如下位置：

```
/etc/ansible/group_vars/raleigh # can optionally end in '.yml', '.yaml', or '.json'
/etc/ansible/group_vars/webservers
/etc/ansible/host_vars/foosball
```

例如，如果在你的清单文件中，你的组主机在你的数据中心，并且每隔数据中心使用它的自有NTP服务器及数据库服务器，你可以创建一个文件称为`/etc/ansible/group_vars/raleigh`去保存`raleigh`组的变量：

```
---
ntp_server: acme.example.org
database_server: storage.example.org
```

你还可以创建以您的组或主机命名的目录。Ansible将会读取这些目录下所有的文件依照字典的顺序。如：

```
/etc/ansible/group_vars/raleigh/db_settings
/etc/ansible/group_vars/raleigh/cluster_settings
```

所有的在`raleigh`组的主机都可以使用这些文件中定义的变量。当单个文件变得太大时，或者当您想在一些组变量上使用Ansible Vault时，这对于保持变量的组织非常有用。

对于`ansible-playbook`，你也可以添加`group_vars/`和`host_vars/`目录到你的playbook目录。其他的Ansible命令（例如：`ansible`,``ansible-console`,等等）仅会在清单目录中寻找`group_vars/`和`host_vars/`目录。如果你想其他的命令可以加载playbook目录下的组和主机变量，你必须在命令行中提供`--playbook-dir`选项。如果你同时从playbook目录和清单目录加载清单文件，在playbook目录的变量将会覆盖清单目录中所定义的变量集。

使用使用版本控制来维护清单文件及变量，是一个很好的跟踪你清单和主机变量变更的方法。



**变量是如何合并的**

<div><a name="var_merged"></a></div>

默认情况下，在一个演义运行前，特定主机变量将会进行合并/扁平化，这样可以让Ansible更专注于主机和任务。所以分组在清单和主机匹配外是不存在的。默认，Ansible将改写为组或主机定义的变量。改写的顺序为从低到高。

- 所有组（为所有分组的父组）

- 父组

- 子组

- 主机

默认Ansible合并相同父/子水平的所有组，通过ASCII的顺序，并且变量来自于最后的组加载并覆盖变量来自于前面的组。例如一个`a_group`将和`b_group`合并，并且`b_group`的匹配的相同的变量值将会覆盖来自于`a_group`的变量值。

注意：Ansible合并来自不同来源的变量，并根据一组规则对某些变量应用优先级。例如，在清单中较高位置的变量可以覆盖在清单中较低位置的变量。参看[Variable precedence: Where should I put a variable?](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#ansible-variable-precedence) 

你可以通过设置组变量`ansible_group_priority`去变更这个行为。为改变相同水平上的组的合并顺序（在父/子顺序解析后），数字越大，越后被合并，并給予的优先级别更高。这个默认的变量值为`1`例如：

```
a_group:
  vars:
    testvar: a
    ansible_group_priority: 10
b_group:
  vars:
    testvar: b
```

在这个例子中，如果两个组都有相同的优先级，结果通常是这样`testvar==b`但是使用上面的ansible_group_priority后，`a_group`的优先级更高，那么结果将会是`testvar==a`

注意：`ansible_group_priority`仅只能在清单源中设置，并不能在group_vars中设置，因为这个变量用于控制group_vars的加载。



**管理清单变量的加载顺序**
<div><a name="mgr_inv_var_order"></a></div>

当使用多个清单来源时，请留意任何冲突的变量将会依照上诉的变量合并规则来解析。你可以控制变量合并的顺序在清单源中取得你所需的变量值。

当你在命令行中传递多个清单源时，Ansible合并变量，并以你提交的清单源的顺序。例如：如果`[all:vars]`在研发的阶段性的清单表中定义`myvar=1`并且在生产的清单表中定义`myvar=2`那么：

- 传递`-i staging -i production`参数去运行Playbook那么`myvar=2`
- 传递`-i production -i staging`参数去运行Playbook那么`myvar=1`

当你将多个不同的清单源放在一个目录中，Ansible将会以ASCII顺序合并它们。你可以通过给文件添加前缀来控制文件的合并顺序：

```
inventory/
  01-openstack.yml          # configure inventory plugin to get hosts from Openstack cloud
  02-dynamic-inventory.py   # add additional hosts with dynamic inventory script
  03-static-inventory       # add static hosts
  group_vars/
    all.yml                 # assign variables to all hosts
```

例如，如果`01-openstack.yml`为所有组`all`定义变量`myvar=1`,`02-dynamic-inventory.py`定义`myvar=2`,并且`03-static-inventory`定义`myvar=3`，那么playbook将会以`myvar=3`运行。



**连接到主机：行为清单参数**
<div><a name="conn_2_host"></a></div>

如上所述，设置如下的变量值可以控制Ansible如何跟远程的主机交互。

主机连接：

注意：Ansible并不提供用户跟ssh进程的参数指定及其他的交互配置，如果需要这些交互内容，那么请使用`ssh-agent`来完成。

**ansible_connection**

到主机的连接类型。这个名称可以为Ansible连接插件的任何连接插件名。SSH协议类型为`ssh`或者`paramiko`默认为ssh.

为所有连接类型的全局性参数：

**ansible_host**

连接到的主机名称,如果跟你给出的别名不一样的话，如果使用定义的方式来给出，那么不要设置它的值依赖于`inventory_hostname`

**ansible_port**

连接端口号

**ansible_user**

连接的用户名

**ansible_password**

连接的密码，不要使用明文来保存它

为SSH协议指定的连接参数：

**ansible_ssh_private_key_file**

如果你不想使用ssh agent，私钥文件

**ansible_ssh_common_args**

这个设定常附加给sftp，scp，ssh命令行。对于给一个确定的主机或者组配置`proxycommand`非常有用。

**ansible_sftp_extra_args**

此设置总是附加到默认的sftp命令行

**ansible_scp_extra_args**

此设置总是附加到默认的scp命令行

**ansible_ssh_extra_args**

此设置总是附加到默认的ssh命令行

**ansible_ssh_pipelining**

确定是否使用SSH管道，这个配置会覆盖`ansible.cfg`中的`pipelining`设置。

**ansible_ssh_executable (added in version 2.2)**

这个设置将改写使用系统的默认行为。这个配置将改写`ansible.cfg`中的`ssh_connection`下的`ssh_executable`配置

---

特权提升：

**ansible_become**

等同于`ansible_sudo`或`ansible_su`允许强制特权提升

**ansible_become_method**

允许设置特权提升的方法

**ansible_become_user**

等同于`ansible_sudo_user`或`ansible_su_user`,允许你设置通过特权提升成为的用户

**ansible_become_password**

等同于`ansible_sudo_password`或`ansible_su_password`允许你设置特权提升的密码（不要使用明文存储密码,使用保险库）

**ansible_become_exe**

等同于`ansible_sudo_exe`或`ansible_su_exe`允许你设置特权提升的可执行方法

**ansible_become_flags**

等同于`ansible_sudo_flags`或`ansible_su_flags`允许你去设置传输给特权提升的方法的标记。这个也可以在`ansible.cfg`的`privilege_escalation`下的`become_flags`进行全局设置。

---

远程主机变量参数：

**ansible_shell_type**

目标系统的shell类型。你不应该使用此配置，直到你设置`ansible_shell_executable`为一个非Bourne（sh）兼容shell。默认，命令使用`sh`风格语法来格式化。设置此参数项为`csh`或者`fish`将会让在目标系统上的命令执行使用所设定shell的风格。

**ansible_python_interpreter**

目标系统的Python路径。这个选项对于有多于一个Python或者python位置不在/usr/bin/python如*BSD或者/usr/bin/python是一个非2.x版本的python非常有用。我们不使用/usr/bin/env机制因为它需要将远程用户的路径设置为正确，并且还假设python可执行文件名为python，因为某些时候可执行文件可能是python2.6

**ansible\_*\_interpreter**

适用于ruby或perl等任何语言，工作方式与ansible_python_interpreter类似。这将替换将在该主机上运行的大量模块。

版本2.1的新功能

**ansible_shell_executable**

这个将设置在目标机器上的ansible控制节点所使用的shell。此设置覆盖`ansible.cfg`上的`executable`参数，默认该参数为/bin/sh

主机文件例子：

```
some_host         ansible_port=2222     ansible_user=manager
aws_host          ansible_ssh_private_key_file=/home/example/.ssh/aws.pem
freebsd_host      ansible_python_interpreter=/usr/local/bin/python
ruby_module_host  ansible_ruby_interpreter=/usr/bin/ruby.1.9.3
```



**非SSH连接类型**

<div><a name="non_ssh"></a></div>

如前一节所述，Ansible通过SSH执行playbook，但它不限于这种连接类型。使用特定于主机的参数`ansible_connection=<connector>`可以更改连接类型。有关可用插件和示例的完整列表，请参见插件列表。



**清单创建例子**

<div><a name="example"></a></div>

**例子：每个环境一个资源清单**

如果你需要管理多个环境，有时候明智的做法是每份清单只定义一个环境的主机。这样一来，你就不太可能在想要更新“预发布”服务器时，不小心更改“测试”环境中节点的状态了。如：

`inventory_test`资源清单文件：

```
[dbservers]
db01.test.example.com
db02.test.example.com

[appservers]
app01.test.example.com
app02.test.example.com
app03.test.example.com
```

上面的文件仅包含了测试环境的主机，下面的文件定义“预发布”环境的主机名为`inventory_staging`

```
[dbservers]
db01.staging.example.com
db02.staging.example.com

[appservers]
app01.staging.example.com
app02.staging.example.com
app03.staging.example.com
```

如：附加一个playbook名为`site.yml`到测试环境的所有的app服务器中，可以使用如何的指令：

```
ansible-playbook -i inventory_test -l appservers site.yml
```



**例子：依照功能分组**

<div><a name="g_by_fun"></a></div>

在前一节中，你已经看到了一个使用组对具有相同功能的主机进行集群的示例，这个可以让你可以进行例如在playbook定义防火墙规则，或者仅影响数据库服务器的角色：

```
- hosts: dbservers
  tasks:
  - name: Allow access from 10.0.0.1
    ansible.builtin.iptables:
      chain: INPUT
      jump: ACCEPT
      source: 10.0.0.1
```



**例子：依照位置分组**

<div><a name="g_by_loc"></a></div>

其他的任务可能是聚焦于一个确认的主机位于那个位置。假设`db01.test.example.com`和`app01.test.example.com`两主机位于DC1，并且`db02.test.example.com`位于DC2：

```
[dc1]
db01.test.example.com
app01.test.example.com

[dc2]
db02.test.example.com
```

