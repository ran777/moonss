所有配置在centos6下进行


<!-- toc orderedList:0 depthFrom:1 depthTo:6 -->

* [增加sudo用户](#增加sudo用户)
* [安装开发环境](#安装开发环境)
* [python安装和配置](#python安装和配置)
* [vim安装和配置](#vim安装和配置)
* [ruby安装配置](#ruby安装配置)
* [Nodejs安装配置](#nodejs安装配置)
* [Nginx安装配置](#nginx安装配置)
* [Jekyll个人博客的安装配置](#jekyll个人博客的安装配置)
* [Jekyll文章发布](#jekyll文章发布)

<!-- tocstop -->


## 增加sudo用户

在root身份下，添加用户ran到root用户组，并设置密码

```bash
adduser ran -g root
passwd ran
```

由于centos系统中不能通过命令将用户添加到sudoer用户组，故手动添加。

编辑/etc/sudoers

```bash
vi /etc/sudoers
```

在"root ALL=(ALL)  ALL"一行下添加：

```vim
ran ALL=(ALL)   ALL
```

键入命令":wq!"，强制保存退出。

## 安装开发环境

更新yum，安装Development tools、wget

```bash
sudo yum update
sudo yum groupinstall 'Development Tools'
sudo yum install wget
```


## python安装和配置

linux下不同程序需要不同版本的python。centos6中的yum使用python2.6，jekyll要求python2.7，个人编程偏向使用python3。所以，使用python环境管理器——conda会更方便。

1. 通过打包好python3和各种常用第三方库的Anaconda安装Python3环境

    ```bash
    wget https://repo.continuum.io/archive/Anaconda3-4.4.0-Linux-x86_64.sh
    bash Anaconda3-4.4.0-Linux-x86_64.sh
    . .bashrc
    ```

2. 使用conda安装名为"py2"的python2.7环境

    ```bash
    conda create --name py2 python=2
    ```

由此，python2.6、python2.7、python3的环境就配置好了。切换不同的环境可参考下面的命令。

```bash
# 查看已安装环境
conda info --envs

# 更改环境
source activate <envname>

# 更改环境为默认环境，这里会切换回python3
source deactivate <envname>
```


## vim安装和配置

>Obtain Vim sources with a git client (recommended).

既然官网推荐github安装，那就git吧

```bash
git clone https://github.com/vim/vim.git
cd vim/src
# vim依赖ncurses-devel，先安装它
sudo yum install ncurses-devel
# 打开vim的python support
./configure --enable-pythoninterp=yes --enable-python3interp=yes
make && sudo make install
vim
```

进行一些基本配置

```bash
vim ~/.vimrc
```

在.vimrc中编辑如下
```vim
"显示行号
set nu

"tab缩进
set tabstop=4
set shiftwidth=4
set softtabstop=4
set expandtab
set smarttab

"自动对齐
set autoindent

"智能缩进
set smartindent

"高亮搜索
set hlsearch

"显示语法
syntax enable
```

## ruby安装配置

类似conda，rvm可以用来管理不同版本的ruby，将ruby置于用户权限下。这里使用rvm安装配置ruby。

安装rvm
```bash
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
\curl -sSL https://get.rvm.io | bash -s stable
source ~/.bashrc
source ~/.bash_profile
source ~/.profile
```

列出已知的 Ruby 版本

```bash
rvm list known
```

安装最新版本的 Ruby

```bash
rvm install ruby
```


1.9以后的ruby已经包含了RubyGems，so安装完毕。

## Nodejs安装配置

下载，解压到~/opt目录下
```bash
wget https://nodejs.org/dist/v6.11.0/node-v6.11.0-linux-x86.tar.xz
tar xf node* -C ~/opt/
mv opt/node* opt/nodejs
```

编辑环境变量~/.bashrc

```vim
NODEJS_HOME="/home/ran/opt/nodejs"
export PATH="$NODEJS_HOME/bin:$NODEJS_HOME/lib:$NODEJS_HOME/share:$PATH"
```

刷新环境变量

```bash
. ~/.bashrc
```

## Nginx安装配置

centos6的yum源中没有Nginx，要手动添加。编辑/etc/yum.repos.d/nginx.repo如下

```bash
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

安装nginx

```bash
sudo yum install nginx
```

测试
```bash
sudo service nginx start
```
ok


## Jekyll个人博客的安装配置

Jekyll安装有以下依赖：
- Ruby（including development headers, Jekyll 2 需要 v1.9.3 及以上版本，Jekyll 3 需要 v2 及以上版本）
- RubyGems
- Linux, Unix, or Mac OS X
- NodeJS, 或其他 JavaScript 运行环境（Jekyll 2 或更早版本需要 CoffeeScript 支持）。
- Python 2.7（Jekyll 2 或更早版本）

通过前面的章节，这些依赖都已满足，准备安装Jekyll吧。首先将python环境切换到python2.7

```bash
source activate py2
```

按照官网doc进行安装，并新建一个blog

```bash
gem install jekyll
# 官网的doc中没提到装bundler，但issue中说到了要装
gem install bundler
bundle install
# 新建一个blog "moonss"
jekyll new moonss
```

接下来是把moonss发布出去，这就要解决一个访问权限问题：Nginx需要访问moonss，但出于安全考虑，不应让Nginx访问到ran目录下的其他文件。

我的策略是通过用户组权限解决。添加一个用户www，由该用户存放moonss；通过组，让用户ran拥有访问www的权限，让用户www拥有访问moonss的权限

```bash
sudo adduser www -g www
sudo usermod -a -G www ran
sudo chmod 770 -R /home/www
sudo chgrp -R www moonss
sudo mv moonss /home/www/
ln -s /home/www/moonss/ /home/ran/
```

文件权限设好了，接着设置nginx的启动用户，端口，网站根目录位置

修改/etc/nginx/nginx.conf

```
user    www;

...
http {

    ...

   #gzip  on;

    server {
        listen       8080;
        server_name  localhost;

        location / {
            root   /home/www/moonss/_site;
            index  index.html index.htm;
        }
    }

    ...

}

```

这样，就可以在ran目录下写文章，通过Jekyll生成静态网页到网站根目录下了。

测试下，开启nginx服务，在浏览器中访问moonss

```bash
sudo service nginx start
```

ok，下一节写一篇文章，并把它发布出去。

## Jekyll文章发布
 /home/ran/opt/anaconda/lib/libpython3.6m.so
Found Python headers folder: /home/ran/opt/anaconda/include/python3.6m
