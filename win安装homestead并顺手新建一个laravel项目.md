## 一. 安装 vagrant
下载，安装，略
url：https://www.vagrantup.com/downloads.html

## 二. 安装 virtualbox
下载，安装，略
url：https://www.virtualbox.org/wiki/Downloads

## 三. 添加 laravel/homestead

这一步骤中，存在两种操作方式

· 命令行万岁

直接使用命令 `vagrant box add laravel/homestead` 来添加 laravel/homestead

但鉴于大陆网络，怕是不得行，当然也可以挂着 VPN 来执行上面的命令。或者曲线救国，得到 homestead 的 virtualbox 文件，来手动添加这一 box 文件

· 曲线救国

命令行执行 `vagrant box add laravel/homestead`

在出现 box: Downloading: https://vagrantcloud.com/laravel/boxes/homestead/versions/5.2.0/providers/virtualbox.box 时用 ctrl+c 退出添加，其实不退出也可以，反正会下载失败导致退出。目的是得到这一 virtualbox.box 文件的下载路径

然后拿着 https://vagrantcloud.com/laravel/boxes/homestead/versions/5.2.0/providers/virtualbox.box 用别的下载软件下下来，命名为 homestead-virtualbox-5.2.0.box 好了，省得分不清版本或者不知道是谁的 virtualbox

创建一个 metadata.json 文件，内容为

```
{
    "name": "laravel/homestead",
    "versions": 
    [
        {
            "version": "5.2.0",
            "providers": [
                {
                  "name": "virtualbox",
                  "url": "homestead-virtualbox-5.2.0.box"
                }
            ]
        }
    ]
}
```

保存在与 homestead-virtualbox-5.2.0.box 同一目录下，当然 url 也可以改成绝对地址或者相对地址，那样 metadata.json 对应着保存也成。然后命令行在目录下执行 `vagrant box add metadata.json`

当看到 `==> box: Successfully added box 'laravel/homestead' (v5.2.0) for 'virtualbox'!` 的时候，那么就成功了

可以用 `vagrant box list` 来看看已有的 box

## 四. 下载 Homestead 项目

找个好地方 `git clone https://github.com/laravel/homestead.git Homestead` 一波就可以，肥肠简单

进入得到的 Homestead 文件夹，一条 `bash init.sh` 命令走起来后，就会得到至关重要的 Homestead.yaml 文件

## 五. 生成密钥对

`ssh-keygen -t rsa` 就很好，当然好像大佬博客里更喜欢 ssh-keygen -t rsa -C "your_email@example.com" ，可能这样有标识更好区分吧

生成密钥过程中，会询问 `Enter file in which to save the key (/c/Users/UserName/.ssh/id_rsa):` ，不修改的话回车即可

还会问 `Enter passphrase (empty for no passphrase):` ，即要不要对密钥对加一个密码，各凭喜好，不需要的话直接回车即可

## 六. 修改 Homestead.yaml 文件

此处略去很多字，哎还是写一写好了。

默认文件长下面这样，其中 ip, memory, cpus, provider 部分应该是讲虚拟机配置吧， authorize 和 keys 部分配置是上一步骤中生成的密钥对，然后 folders 是说虚拟机的文件夹映射， sites 是浏览器的访问映射， database 则是 sites 对应 - 的数据库

也就是说进入虚拟机后 /home/vagrant/code 对应的就是本地 ~/code ，浏览器访问 homestead.test 时入口是虚拟机里的 /home/vagrant/code/public

然后 homestead.test 所访问 laravel 项目用的数据库就是 homestead

```
---
ip: "192.168.10.10"
memory: 2048
cpus: 1
provider: virtualbox

authorize: ~/.ssh/id_rsa.pub

keys:
    - ~/.ssh/id_rsa

folders:
    - map: ~/code
      to: /home/vagrant/code

sites:
    - map: homestead.test
      to: /home/vagrant/code/public

databases:
    - homestead

# blackfire:
#     - id: foo
#       token: bar
#       client-id: foo
#       client-token: bar

# ports:
#     - send: 50000
#       to: 5000
#     - send: 7777
#       to: 777
#       protocol: udp


```

下面来把 folders 下的 map 改一改

```
folders:
    - map: E:/laravel_code
      to: /home/vagrant/code
```

把代码文件位置和虚拟机中共享文件位置设成 E 盘下的 laravel_code

再把 sites 下的 to 改一改

```
sites:
    - map: homestead.test
      to: /home/vagrant/code/test/public
```

为接下来创建名为 test 的 laravel 项目做一下准备

## 七. 启动 vagrant

在 Homestead 目录下执行 `vagrant up` ，闪过一大堆输出后，大概是成功了的样子，再一条 `vagrant ssh` 下去连接虚拟机，看到

```
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-112-generic x86_64)
```

大概就是成功启动了！可输入 `exit` 退出虚拟机。至于停止 vagrant ， `vagrant halt` 了解一下，哦对如果更新了 Homestead.yaml 文件，需要使用 `vagrant provision` 来生效配置。

## 八. 创建名为 test 的 laravel 项目

首先一个 `vagrant up` 登上虚拟机，然后准备用 composer 来创建一个 laravel 项目

```
composer create-project laravel/laravel test --prefer-dist
```

然而， composer 的大陆速度堪忧，看来还是得改个源比较好

```
 composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

上面这条改 composer 源的命令请了解一下，你甚至还可以通过 `composer config -l -g ` 来看看你的源改没改成功

现在就可以用浏览器访问一下 192.168.10.10 来看看可爱的 laravel 默认界面，至于为什么还不能用 homestead.test 访问，是因为还没改 windows 系统里万恶的 hosts 文件

## 九. 改一改 hosts 文件

一般来说， windows 的 hosts 文件在目录 C:\Windows\System32\drivers\etc 下

然而直接在这个目录下修改 hosts 文件会大麻烦，所以先把 hosts 文件拷到桌面上，用随便什么编辑器（记事本也口以）打开

在末尾添加一行

```
192.168.10.10  homestead.test
```

然后保存，拷回 C:\Windows\System32\drivers\etc ，然后会需要权限，给给给就行

这个时候，浏览器再输入 homestead.test 就能访问了，完美

END