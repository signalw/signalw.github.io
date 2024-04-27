---
layout: post
title: 在CentOS上用rbenv安装Ruby
---

很久没有碰Ruby了，最近想把GitHub上的一个项目clone到本地build时，才发现新装的CentOS上连Ruby都没有，于是就有了这篇文章。

Ruby官网上列了4种不同的安装方法：

- 使用系统软件包管理系统，如Ubuntu的apt-get，CentOS的yum，macOS上常用的homebrew等
- 使用Ruby专门的安装程序，如ruby-build，ruby-install，windows上的exe执行文件等
- 使用rbenv，RVM等支持安装及切换不同Ruby版本的管理工具
- 下载ruby tarball后手动编译安装

如果你有在多个不同版本的Ruby间切换的需求，rbenv应该是个不错的选择，而且从各方评价来看，rbenv似乎要高于RVM。

这是rbenv的源代码和官网：https://github.com/rbenv/rbenv，有非常详细的说明。

我的机器是CentOS 7，但是其他Linux的版本安装流程应该基本一样。

第一步把源代码clone到本地`~/.rbenv`目录中
```bash
$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv
Cloning into '/home/centos/.rbenv'...
remote: Counting objects: 2714, done.
remote: Total 2714 (delta 0), reused 0 (delta 0), pack-reused 2714
Receiving objects: 100% (2714/2714), 506.40 KiB | 0 bytes/s, done.
Resolving deltas: 100% (1700/1700), done.
```

这步非必需：编译bash extension，主要是为了加速rbenv运行，可能会报错，可以忽略。我发现我连gcc都没安装，安装后并没有报错
```bash
$ cd ~/.rbenv && src/configure && make -C src
warning: gcc not found; using CC=cc
aborted: compiler not found: cc
$ sudo yum install gcc
$ cd ~/.rbenv && src/configure && make -C src
make: Entering directory `/home/centos/.rbenv/src'
gcc -fPIC     -c -o realpath.o realpath.c
gcc -shared -Wl,-soname,../libexec/rbenv-realpath.dylib  -o ../libexec/rbenv-realpath.dylib realpath.o
make: Leaving directory `/home/centos/.rbenv/src'
```

第二步添加环境变量。注意，如果是Ubuntu的话应该编辑`.bashrc`而不是`.bash_profile`
```bash
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
```

第三步运行初始化，并根据提示把eval那个语句添加到`.bash_profile`
```bash
$ ~/.rbenv/bin/rbenv init
# Load rbenv automatically by appending
# the following to ~/.bash_profile:

eval "$(rbenv init -)"

$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
```

第四步重启shell，或者直接在当前窗口运行`source ~/.bash_profile`以让刚才添加的环境变量等生效
```bash
source ~/.bash_profile
```

第五步用`rbenv-doctor`验证安装是否成功。可以看到有一条指令未安装的提醒，我们在下一步就安装。
```bash
$ curl -fsSL https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-doctor | bash
Checking for `rbenv' in PATH: /home/centos/.rbenv/bin/rbenv
Checking for rbenv shims in PATH: OK
Checking `rbenv install' support: which: no rbenv-install in (/home/centos/.rbenv/shims:/home/centos/.rbenv/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/centos/.local/bin:/home/centos/bin:/home/centos/.local/bin:/home/centos/bin)
bash: line 114: : command not found
 ()
Counting installed Ruby versions: none
  There aren't any Ruby versions installed under `/home/centos/.rbenv/versions'.
  You can install Ruby versions like so: rbenv install 2.2.4
Checking RubyGems settings: OK
Auditing installed plugins: OK
```

第六步安装ruby-build
```bash
$ mkdir -p "$(rbenv root)"/plugins
$ git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build
Cloning into '/home/centos/.rbenv/plugins/ruby-build'...
remote: Counting objects: 8867, done.
remote: Compressing objects: 100% (19/19), done.
remote: Total 8867 (delta 8), reused 18 (delta 2), pack-reused 8843
Receiving objects: 100% (8867/8867), 1.86 MiB | 0 bytes/s, done.
Resolving deltas: 100% (5684/5684), done.
```
从GitHub clone到本地后并不需要额外操作。下面我们再来验证一下是否安装成功
```bash
$ curl -fsSL https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-doctor | bash
Checking for `rbenv' in PATH: /home/centos/.rbenv/bin/rbenv
Checking for rbenv shims in PATH: OK
Checking `rbenv install' support: which: no rbenv-install in (/home/centos/.rbenv/shims:/home/centos/.rbenv/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/centos/.local/bin:/home/centos/bin:/home/centos/.local/bin:/home/centos/bin)
/home/centos/.rbenv/plugins/ruby-build/bin/rbenv-install (ruby-build 20180601)
Counting installed Ruby versions: none
  There aren't any Ruby versions installed under `/home/centos/.rbenv/versions'.
  You can install Ruby versions like so: rbenv install 2.2.4
Checking RubyGems settings: OK
Auditing installed plugins: OK
```

至此rbenv就安装完成啦！下面我们来随心所欲地安装和切换不同版本的Ruby！使用`rbenv install -l`可以列出所有可选的Ruby版本。列表太长就不把输出贴出来了。

先安装一个2.3.0吧。但是，等了半天又报错了…
```bash
$ rbenv install 2.3.0
Downloading ruby-2.3.0.tar.bz2...
-> https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.0.tar.bz2
Installing ruby-2.3.0...

BUILD FAILED (CentOS Linux 7 using ruby-build 20180601)

Inspect or clean up the working tree at /tmp/ruby-build.20180602035853.9426
Results logged to /tmp/ruby-build.20180602035853.9426.log

Last 10 log lines:
The Ruby openssl extension was not compiled.
The Ruby readline extension was not compiled.
The Ruby zlib extension was not compiled.
ERROR: Ruby install aborted due to missing extensions
Try running `yum install -y openssl-devel readline-devel zlib-devel` to fetch missing dependencies.

Configure options used:
  --prefix=/home/centos/.rbenv/versions/2.3.0
  LDFLAGS=-L/home/centos/.rbenv/versions/2.3.0/lib
  CPPFLAGS=-I/home/centos/.rbenv/versions/2.3.0/include
```
原来是缺少一些依赖包，这些都安装完后再试试
```bash
$ sudo yum install -y openssl-devel readline-devel zlib-devel
$ rbenv install 2.3.0
Downloading ruby-2.3.0.tar.bz2...
-> https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.0.tar.bz2
Installing ruby-2.3.0...
Installed ruby-2.3.0 to /home/centos/.rbenv/versions/2.3.0
```

耐心等待几分钟后终于成功！顺手再装2个版本吧。然后使用`rbenv versions`便会列出本地安装的所有Ruby版本，如果要切换到某个特定版本就用`rbenv global <version number>`即可
```bash
$ rbenv install 2.5.0
Downloading ruby-2.5.0.tar.bz2...
-> https://cache.ruby-lang.org/pub/ruby/2.5/ruby-2.5.0.tar.bz2
Installing ruby-2.5.0...
Installed ruby-2.5.0 to /home/centos/.rbenv/versions/2.5.0
$ rbenv install 2.4.0
Downloading ruby-2.4.0.tar.bz2...
-> https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.0.tar.bz2
Installing ruby-2.4.0...
Installed ruby-2.4.0 to /home/centos/.rbenv/versions/2.4.0
$ rbenv global 2.3.0
$ rbenv versions
* 2.3.0 (set by /home/centos/.rbenv/version)
  2.4.0
  2.5.0
$ ruby -v
ruby 2.3.0p0 (2015-12-25 revision 53290) [x86_64-linux]
```

我在安装Ruby稳定版的过程中并没有遇到问题，但是在安装非稳定版本和jruby时都报错了，估计是新系统还缺少很多依赖包导致的。

关于卸载Ruby和卸载rbenv在官方的documentation里都有详细说明，非常简单直接，这里就不一一展开了。

视频demo: https://v.youku.com/v_show/id_XMzY1NzYxMTkyOA==.html