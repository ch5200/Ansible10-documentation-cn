# Ansible 10 中文文档

## Ansible使用指南

### 创建Ansible资源清单

欢迎使用Ansible资源清单创建指南。在Ansible中，所谓资源清单（或者叫做资源仓库）是一个将被Ansible配置管理或部署软件的受管的节点或主机列表。本指南将向你介绍资源清单，涵盖如下主题：

- 创建资源清单，以便于跟踪你需要自动处理的服务器和资源列表。
- 使用动态资源清单，跟踪云服务的服务器及设备启动及关闭。
- 使用模式去自动化资源清单中指定的子集。
- 扩展和改进Ansible跟你的资源清单的连接方式。
- [如何创建你的资源清单](#Ansible10CNDoc-使用指南-如何创建资源清单.md)
  - 资源清单的基础知识：格式，主机及主机组
  - 传递多个清单源
  - 在目录中组织资源清单
  - 在资源清单中添加变量
  - 给一个机器分配变量：主机变量
  - 使用INI格式定义变量
  - 给多个机器分配变量：组变量
  - 组织主机或主机组的变量
  - 变量是如何合并的
  - 连接到主机：行为清单参数
  - 资源清单创建例子
- 使用动态资源清单
  - 资源清单脚本例子：Cobbler
  - 其他资源清单脚本
  - 使用资源清单目录及多个资源清单源
  - 动态组中的静态组
- 使用模式指定主机或主机组
  - 使用模式
  - 常用模式
  - 模式的局限性
  - 模式处理顺序
  - 高级模式选项
  - 模式和临时指令
  - 模式和ansible-playbook 标记
- 连接方法和详情
  - [ControlPersist and paramiko](https://docs.ansible.com/ansible/latest/inventory_guide/connection_details.html#controlpersist-and-paramiko)
  - [Setting a remote user](https://docs.ansible.com/ansible/latest/inventory_guide/connection_details.html#setting-a-remote-user)
  - [Setting up SSH keys](https://docs.ansible.com/ansible/latest/inventory_guide/connection_details.html#setting-up-ssh-keys)
  - [Running against localhost](https://docs.ansible.com/ansible/latest/inventory_guide/connection_details.html#running-against-localhost)
  - [Managing host key checking](https://docs.ansible.com/ansible/latest/inventory_guide/connection_details.html#managing-host-key-checking)
  - [Other connection methods](https://docs.ansible.com/ansible/latest/inventory_guide/connection_details.html#other-connection-methods)