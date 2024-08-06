# Ansible 10 中文文档

## Ansible 安装指南

### 配置Ansible

**主题**

- 配置Ansible
  - [配置文件](#config_file)
    - [获取最新的配置](#get_newest_cfg)
  - [环境配置](#env_cfg)
  - [命令行选项](#cmd_opt)

本章节主题主要描述如何控制Ansible的配置。

---

**配置文件**

<div><a name="config_file"></a></div>

Ansible中的某些设置可以通过配置文件(`Ansible .cfg`)进行调整。对于大多数用户来说，现有的配置应该已经足够了，但是您可能会因为某些原因想要更改它们。

参考文档中列出了搜索配置文件的路径。

- `ANSIBLE_CONFIG` (环境变量，如果有设置 )
- `ansible.cfg` (当前目录下)
- `~/.ansible.cfg` (home目录下)
- `/etc/ansible/ansible.cfg`



**获取最新的配置**

<div><a name="get_newest_cfg"></a></div>

如果从包管理器安装Ansible，那么最新的`ansible.cfg`文件应该在`/etc/ansible`文件夹下，可能以.rpmnew扩展名或其他的扩展名存在。

如果你从`pip`或者源码中安装Ansible，那么你可能需要手工创建这个文件，以便于覆盖Ansible的某些默认设置。

你可以生成Ansible的配置文件`ansible.cfg`,其中里面包含了所有的默认配置项：

```shell
$ ansible-config init --disabled > ansible.cfg
```

如果想要包含所有插件的更完整的配置，可以用如下的指令生成：

```shell
$ ansible-config init --disabled -t all > ansible.cfg
```

有关更多细节和可用配置的完整列表，请访问[configuration_settings](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings)

你可以使用`ansible-config`命令行工具来列出你的变量选项，并检查当前值



**环境配置**

<div><a name="env_cfg"></a></div>

Ansible同时允许使用环境变量来进行配置设定。一旦环境变量被设置，那么该变量将覆盖从配置文件中加载的相关任何配置。你可以从以下的链接获取完整的环境变量列表：

- [Ansible Configuration Settings](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings): 用于配置核心功能
- [Index of all Collection Environment Variables](https://docs.ansible.com/ansible/latest/collections/environment_variables.html#list-of-collection-env-vars): 用于配置集合中的插件



**命令行选项**

<div><a name="cmd_opt"></a></div>

并非所有配置选项都出现在命令行中,仅仅是那些常见的有用的选项。在命令行中设置某些参数选项，将会传递给程序，并覆盖任何配置文件和环境变量的相关配置。

完整的选项请参看 [ansible-playbook](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html#ansible-playbook) and [ansible](https://docs.ansible.com/ansible/latest/cli/ansible.html#ansible).
