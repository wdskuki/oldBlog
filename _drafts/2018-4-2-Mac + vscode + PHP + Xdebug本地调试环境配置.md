

[TOC]

##安装homebrew

Mac上用的是[homebrew](https://brew.sh/)来进行包的安装和管理，这里先安装[homebrew](https://brew.sh/) (已经安装的可以此步骤)

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

建议开代理安装，否则安装非常慢。

安装完[homebrew](https://brew.sh/)，替换homebrew更新源。我这里替换为[中科大的源](https://lug.ustc.edu.cn/wiki/mirrors/help/brew.git)。

```shell
替换brew.git:
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git

替换homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
```




##安装PHP7.2

##安装Xdebug

###下载

安装Xdebug可以参考[官网流程](https://xdebug.org/wizard.php)。通过`php -i`获得`phpinfo()`的信息，然后将这个信息倒入官网里获得对应Xdebug的版本。

### 安装



##vscode配置



##测试





## 参考文章

1. [Mac 通过phpize安装xdebug](https://blog.csdn.net/coderDerek/article/details/75208418)
2. ​

