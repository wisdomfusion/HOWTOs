# How to Install Multiple Versions of Perl on Mac OS X

## 0. 安装 Perlbrew

类似 OS X 包管理器 [Homebrew](http://brew.sh/)，[Perlbrew](https://perlbrew.pl/) 是 OS X 或 Linux 下多版本 Perl 的管理器。安装和使用都极其简便，下面我们看看 Perlbrew 是如何做到多版本 Perl 的安装和控制的。

首先在 OS X 的终端中执行如下 shell 脚本安装 Perlbrew：

```sh
\curl -L http://install.perlbrew.pl | bash
```

再简单不过了，安装后可以用 `perlbrew version` 查看一下当然的版本：

    $ perlbrew version
    /Users/huzhifei/perl5/perlbrew/bin/perlbrew  - App::perlbrew/0.75

使用 `perlbrew help` 可以查看其文档，快速撑握其用法。

这里列出了日常的一些常用命令和更新 perlbrew 和安装管理不同版本的 Perl：

## 1. 查看可安装的 Perl 版本

```sh
perlbrew available
```

示例输出：

    $ perlbrew available
      perl-5.25.1
    i perl-5.24.0
    i perl-5.22.2
      perl-5.20.3
      perl-5.18.4
      perl-5.16.3
      perl-5.14.4
      perl-5.12.5
      perl-5.10.1
    i perl-5.8.9
      perl-5.6.2
      perl5.005_04
      perl5.004_05
      perl5.003_07

其中前面标有 `i` 的是目前安装的 Perl 版本。

## 2. 安装某个 Perl 版本

```sh
perlbrew install perl-5.24.0
```

## 3. 列出目前安装的 Perl 版本

```sh
perlbrew list
```

示例输出：

    $ perlbrew list
      perl-5.8.9
      perl-5.22.1
    * perl-5.22.2
      perl-5.24.0

其中标有 `*` 的是当前正在使用的 Perl 版本。

## 4. 在已安装的 Perl 版本间切换

```sh
perlbrew switch
```

## 5. 更新当前使用的 Perl 版本

```sh
perlbrew upgrade-perl
```
如当然用的是 `5.14.0` 版，使用该命令可以更新到 `5.14.2`，即更新当然大版本的小版本更新（`5.x.*`）。

## 6. 更新 Perlbrew

```sh
perlbrew self-upgrade
```

## 7. 卸载 Perl

```sh
perlbrew uninstall <name>
```

## 8. 使用系统原有的 Perl

如果想要关闭 Perlbrew 和用它安装的 Perl，执行如下命令在当然 shell 中关闭 Perlbrew：

```sh
perlbrew off
```

执行如下命令，永久关闭 Perlbrew：

```sh
perlbrew switch-off
```

## 9. 针对不同的 Perl 版本进行代码测试

使用 Perlbrew 不只是能安装和管理不同的 Perl 版本，而且还能针对目前安装的 Perl 版本进行代码测试：

```sh
perlbrew exec perl -e 'print "Hello from $]\n"'
```

标例输出：

    perl-5.12.2
    ==========
    Hello word from perl-5.012002
    
    perl-5.13.10
    ==========
    Hello word from perl-5.013010
    
    perl-5.14.0
    ==========
    Hello word from perl-5.014000

更多命令请参看 `perlbrew help`。
