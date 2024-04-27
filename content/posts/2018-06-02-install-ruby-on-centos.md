+++
title = 'Install Ruby on CentOS with rbenv'
date = 2018-06-02T14:04:00-04:00
comments = true
math = false
+++

I haven't touched Ruby for a long time. Recently, when I wanted to clone a project on GitHub and build it locally, I found that Ruby was not even available on the newly installed CentOS. That is the reason why this article is written.

The Ruby official website lists 4 different installation methods:

- Use package management tools, such as Ubuntu's apt-get, CentOS's yum, homebrew commonly used on macOS, etc.
- Use Ruby-specific installation programs, such as ruby-build, ruby-install, .exe files on windows, etc.
- Use rbenv, RVM and other management tools to support the installation and switching on different versions
- Download the Ruby tarball and manually compile and install it


If you have the need to switch between multiple different versions of Ruby, rbenv should be a good choice, and based on various reviews, rbenv seems to be better than RVM, as of this writing.

This is the official website of rbenv: https://github.com/rbenv/rbenv, with its source code and very detailed instructions.

The machine I am using is on CentOS 7, but the installation process should be pretty similar for other Linux distros.

The first step is to clone the source code into the local `~/.rbenv` directory.
```bash
$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv
Cloning into '/home/centos/.rbenv'...
remote: Counting objects: 2714, done.
remote: Total 2714 (delta 0), reused 0 (delta 0), pack-reused 2714
Receiving objects: 100% (2714/2714), 506.40 KiB | 0 bytes/s, done.
Resolving deltas: 100% (1700/1700), done.
```

This step is optional. Compiling the bash extension is mainly to speed up the operation of rbenv. An error may be reported but can be ignored. I found that I didn't even have gcc. No errors were reported after installation.
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

The second step is to add environment variables. Note that if it is Ubuntu, you should edit `.bashrc` instead of `.bash_profile`.
```bash
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
```

The third step is initialization and add the eval statement to `.bash_profile` according to the prompts.
```bash
$ ~/.rbenv/bin/rbenv init
# Load rbenv automatically by appending
# the following to ~/.bash_profile:

eval "$(rbenv init -)"

$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
```

The fourth step is to restart the shell, or run `source ~/.bash_profile` directly in the current session to make the new environment variables take effect.
```bash
source ~/.bash_profile
```

Step 5: Use `rbenv-doctor` to verify whether the installation is successful. You can see a reminder that a command is not installed. We will install it in the next step.
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

Step 6: Install ruby-build
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
No additional operations are required after cloning from GitHub to local. Next, let’s verify whether the installation is successful.
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

At this point, the rbenv installation is complete! Now let's install and switch between different versions of Ruby as we like! Use `rbenv install -l` to list all available Ruby versions. The list is too long so I won't post everything here.

Let's install 2.3.0 first. However, it returns error again...
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
It turns out that some dependency packages are missing. Try again after installing these.
```bash
$ sudo yum install -y openssl-devel readline-devel zlib-devel
$ rbenv install 2.3.0
Downloading ruby-2.3.0.tar.bz2...
-> https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.0.tar.bz2
Installing ruby-2.3.0...
Installed ruby-2.3.0 to /home/centos/.rbenv/versions/2.3.0
```

After a couple of minutes, it finally worked! Let's install 2 more versions. Then use `rbenv versions` to list all locally installed Ruby versions. If you want to switch to a specific version, just use `rbenv global <version number>`.

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

I didn't encounter any problems when installing the stable versions, but when installing the non-stable versions as well as jruby, I ran into errors again. It's probably due to missing dependencies again.

There are detailed instructions on uninstalling Ruby and uninstalling rbenv in the official documentation, so I will not go into details here.
