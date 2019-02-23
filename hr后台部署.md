#### hr后台部署

（<u>vpn建议连接美国节点</u>，环境需要vagrant homestead composer laravel git）

#### 1.克隆项目并建立本地仓库、关联远程仓库

在虚拟机设置的共享文件夹下执行

```
git clone https://git.coding.net/SadCreeper/uestc-hr-server.git hr_server
cd hr_server
git init
git remote add origin https://git.coding.net/SadCreeper/uestc-hr-server.git
//之后会让你输入你的coding账户以及密码，填好后确定即可关联成功
```



#### 2.修改.env文件

进入项目执行 `cp .env.example .env` ，打开 .env 修改如下

```php
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=hrdb
DB_USERNAME=hr
DB_PASSWORD=happy
```

#### 2.1修改homestead.yaml文件、

在sites和databases里添加

- ```
  sites:
  
  - map: hr.server
        to: /home/vagrant/code/hr_server/public
  
  databases:
  
  - hrdb
  ```

#### 2.2修改hosts文件（C:\Windows\System32\drivers\etc)

在末尾处添加

```
192.168.10.10  hr.server
```

注意该行前面不要有#，会被注释掉

#### 3.启动homestead

cmd中执行

```
cd homestead

vagrant up

vagrant provision

vagrant ssh
```

#### 4.下载包

进入项目根目录执行 `composer install`

**可能**出现 没有PHP Idap扩展 的报错,
!(https://github.com/yuzhongduobu/xuoxi/blob/master/1550905480623.png?raw=true)

  Homestead 下 php 安装 ldap 扩展

```
sudo php -v   //先查看版本

sudo apt-get update

sudo apt-get install php7.2-ldap   //安装对应版本的 ldap 扩展

//安装过程中出现是否替换的提示，选择不替换  
```

![1550905536324](https://github.com/yuzhongduobu/xuoxi/blob/master/1550905536324.png?raw=true)

此处选择第一项

```
install the package maintainer's version
```

然后执行

```
composer install
```

#### 5.创建key

```
php artisan key:generate
```

#### 6.移植数据表

```
php artisan migrate
```

**可能**出现移植错误的情况

![1550905834486](https://github.com/yuzhongduobu/xuoxi/blob/master/1550905834486.png?raw=true)

就执行

```
php artisan migrate:fresh --seed

```

#### 6.1安装passport

```
php artisan passport:install
```

#### 6.2补入测试数据（如果执行了前面migrate:fresh --seed命令，该步跳过）

```
php artisan db:seed
```

#### 7.进行开发

在本地cmd进入项目文件夹根目录

```
npm install
npm run watch //实现前端js预览编译
```

如果npm install出现错误，可以试试以下方法，更换为淘宝镜像
![1550906854492](https://github.com/yuzhongduobu/xuoxi/blob/master/1550906854491.png?raw=true)

![1550906854491](https://github.com/yuzhongduobu/xuoxi/blob/master/1550906806910.png?raw=true)


然后再执行npm run watch

在chrome地址栏输入hr.server，看看能否访问成功，如果不行就重启电脑（可能是hosts文件没有生效），

虚拟机只需执行vagrant up和vagrant ssh,本地的项目根目录执行npm run watch
