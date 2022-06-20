# Qing-LKY 的个人博客

## 前言

博客园不知道为什么不能实时更新了。感觉很蛋疼。所以想试着自己搭一个博客。

使用的是 github.io + hexa 的经典组合。暂时选了 next 作为主题。

## 搭建过程

首先是下载 git 和 node.js。之前都下过，只是后来电脑出了点小问题导致缺 dll 文件用不了。所以重新装了一遍。

git 的下载地址是 [git-scm.com](https://git-scm.com/downloads)。

node.js 的下载地址是 [nodejs.cn](http://nodejs.cn/download/)。

如果是初次使用的话需要为 git 配置 user.name 和 user.email。可以为 github 添加 ssh 密钥。

```bash
git config --global user.name "xxx"
git config --global user.email "xxx@xxx"
ssh-keygen -t rsa -C "xxx@xxx"
cat ~/.ssh/id_rsa.pub
```

然后到 github 上添加 ssh key 就行。添加完之后可以 ssh -T git@github.com 试一下。

如果装完 node.js 后测试 npm -v 时报 warn 说什么 local 啊什么 global 什么的，那个可能是由于版本问题导致 -g 无法直接对应到 --location=global 上。你可以这样做：

- 先更新你的 npm 看看是否还有问题。如果还有可以干下面的事情：
- 用管理员权限打开记事本，然后用带管理员权限的记事本去修改 nodejs/npm 和 nodejs/npm.cmd。这只是一个参考，如果你用其它方式修改（比如直接改这两个文件的权限或是用别的软件啊、直接登录管理员啊）都行。
- 把里面 prefix -g 替换成 prefix --location=global。

然后是 hexo 的安装和配置。可参考[官网](https://hexo.io/zh-cn/docs/)。

接下来创一个目录，接下来它就会是我们处理这个博客的工程目录。进去这个目录。安装 hexo。

```bash
npm install -g hexo-cli
echo 'PATH="$PATH:./node_modules/.bin"' >> ~/.profile
hexo init
npm install
```