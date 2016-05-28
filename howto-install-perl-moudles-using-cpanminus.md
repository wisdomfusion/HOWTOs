# How to Install Perl Modules Using App::cpanminus (cpanm)

## 0. 关于 App::cpanminus

多数 Perl 模块的安装会遇到大量的选项和模块依赖的问题，不少刚接触 Perl 的同学，可能会被这个拦路虑挡在门外，本文没有像其他书籍或教程一样给出若干安装方法的选择，而是直接介绍一种最常用最不易出错的方法，让大家简单高效地安装 Perl 模块，让我们专注问题的解决上。这种方法，就是使用 [App::cpanminus](https://metacpan.org/pod/App::cpanminus) 模块，这也是 Perl 官方推荐的模块安装方法。

## 1. 安装 cpanm

App::cpanminus 简称 cpanm，这是因为安装该模块后，会在系统中安装 `cpanm` 的命令，用于 Perl 模块的管理。

meta::cpan 中的模块页面中有关于 App::cpanminus 的[安装说明](https://metacpan.org/pod/App::cpanminus#INSTALLATION)，这里再从实际角度重复说明一下不同平台的安装过程。

**Mac OS X**

一般我们会把 cpanm 和将要安装的 Perl 模块安装在当前用户（非 root 用户）目录下的 `~/perl5/` 文件夹中。运行如下几条命令：

    curl -L https://cpanmin.us | perl - -l ~/perl5 App::cpanminus local::lib
    eval `perl -I ~/perl5/lib/perl5 -Mlocal::lib`
    echo 'eval `perl -I ~/perl5/lib/perl5 -Mlocal::lib`' >> ~/.profile

**Linux**

Linux 平台下安装比较直接，一条命令搞惦：

    curl -L https://cpanmin.us | perl - --sudo App::cpanminus

也可以直接手动下载安装：

    cd ~/bin
    curl -L https://cpanmin.us/ -o cpanm
    chmod +x cpanm

**Windows**

``When I'm on Windows, I use Strawberry Perl'' -- Larry Wall

我在 Windows 平台也使用[大草莓 Perl](http://strawberryperl.com/)，安装后，打开 Perl 命令行或命令提示符，执行 `cpan App::cpanminus` 命令即可快速安装 cpanm。

## 2. cpanm 的一些使用技巧

### 2.1 模块的安装

比如，我想安装 [Mojolicious](http://mojolicious.org/) 这个 Perl Web 框架，执行如下命令：

    cpanm Mojolicious

已安装的模块，该命令会输出当然的模块信息：

    $ cpanm Mojolicious
    Mojolicious is up to date. (6.62)

也可以一次性安装多个模块：

    cpanm AnyEvent Coro DBIx::Class Dancer EV LWP Modern::Perl Mojolicious Moo Smart::Comments Spreadsheet::ParseExcel Spreadsheet::WriteExcel Template Test::More Web::Scraper

安装模块的指定版本：

    cpanm Mojolicious@6.42

另外，个别模块因为如果通过墙外的 URL 进行网络测试，导致无法安装，可以尝试使用 `-n,--notest` 参数，甚至 `-f,--force`。

### 2.2 删除模块

    cpanm -U Mojolicious

### 2.3 更新过期的模块

先安装 App::cpanoutdated 模块：

    cpanm App::cpanoutdated

列出过期的模块：

    cpan-outdated

直接把过期的模块使用管道传给 cpanm 命令，更新安们：

    cpan-outdated | cpanm

### 2.4 指定 CPAN 镜像

有时候默认的官方 CPAN 更新可能比较慢，或是直接更新不了，这个时候 CPAN 镜像就起大作用了，cpanm 命令可以直接指定某个镜像：

    cpanm --sudo --mirror http://mirrors.ustc.edu.cn/CPAN/ --mirror-only

可选的 CPAN 镜像如下：

    ftp://ftp.cuhk.edu.hk/pub/packages/perl/CPAN/
    ftp://mirror.communilink.net/CPAN/
    ftp://mirrors.sohu.com/CPAN/
    ftp://mirrors.ustc.edu.cn/CPAN/
    ftp://mirrors.xmu.edu.cn/CPAN/
    http://cpan.communilink.net/
    http://ftp.cuhk.edu.hk/pub/packages/perl/CPAN/
    http://mirror.qdu.edu.cn/CPAN/
    http://mirrors.163.com/cpan/
    http://mirrors.btte.net/CPAN/
    http://mirrors.devlib.org/cpan/
    http://mirrors.sohu.com/CPAN/
    http://mirrors.ustc.edu.cn/CPAN/
    http://mirrors.xmu.edu.cn/CPAN/

为了方便，可以在 `~/.bashrc` 或 `~/.profile` 中添加该命令的别名，如：

    echo "alias cpanm='cpanm --sudo --mirror http://mirrors.ustc.edu.cn/CPAN/ --mirror-only'" >> ~/.profile
    source ~/.profile


### 2.5 查看模块信息

安装 App::pmodinfo 模块：

    cpanm App::pmodinfo

该模块会安装一个 pmodinfo 命令，使用方法：

    $ pmodinfo -h
    Usage: pmodinfo [options] [Module] [...]
    
        -v --version            Display software version
        -f --full               Turns on the most output
        -h --hash               Show module and version in a hash.
        -l,--local-modules      Display all local modules
        -u,--check-updates      Check updates, compare your local version to cpan.
        -c,--cpan               Show the last version of module in cpan.

也可以直接用 cpanm 命令查看某个模块的信息：

    $ cpanm --info Mojolicious
    SRI/Mojolicious-6.62.tar.gz

### 2.6 更新 cpanm

使用 `cpanm --sudo --self-upgrade` 或 `cpanm --self-upgrade`

    $ cpanm --self-upgrade
    --> Working on App::cpanminus
    Fetching http://www.cpan.org/authors/id/M/MI/MIYAGAWA/App-cpanminus-1.7042.tar.gz ... OK
    Configuring App-cpanminus-1.7042 ... OK
    Building and testing App-cpanminus-1.7042 ... OK
    Successfully installed App-cpanminus-1.7042 (upgraded from 1.7041)
    1 distribution installed

## 3. More

    $ cpanm --help
        Usage: cpanm [options] Module [...]
    
        Options:
    -v,--verbose              Turns on chatty output
      -q,--quiet                Turns off the most output
      --interactive             Turns on interactive configure (required for Task:: modules)
      -f,--force                force install
      -n,--notest               Do not run unit tests
      --test-only               Run tests only, do not install
      -S,--sudo                 sudo to run install commands
      --installdeps             Only install dependencies
      --showdeps                Only display direct dependencies
      --reinstall               Reinstall the distribution even if you already have the latest version installed
      --mirror                  Specify the base URL for the mirror (e.g. http://cpan.cpantesters.org/)
      --mirror-only             Use the mirror's index file instead of the CPAN Meta DB
      -M,--from                 Use only this mirror base URL and its index file
      --prompt                  Prompt when configure/build/test fails
      -l,--local-lib            Specify the install base to install modules
      -L,--local-lib-contained  Specify the install base to install all non-core modules
      --self-contained          Install all non-core modules, even if they're already installed.
      --auto-cleanup            Number of days that cpanm's work directories expire in. Defaults to 7
    
    Commands:
      --self-upgrade            upgrades itself
      --info                    Displays distribution info on CPAN
      --look                    Opens the distribution with your SHELL
      -U,--uninstall            Uninstalls the modules (EXPERIMENTAL)
      -V,--version              Displays software version
    
    Examples:
    
      cpanm Test::More                                          # install Test::More
      cpanm MIYAGAWA/Plack-0.99_05.tar.gz                       # full distribution path
      cpanm http://example.org/LDS/CGI.pm-3.20.tar.gz           # install from URL
      cpanm ~/dists/MyCompany-Enterprise-1.00.tar.gz            # install from a local file
      cpanm --interactive Task::Kensho                          # Configure interactively
      cpanm .                                                   # install from local directory
      cpanm --installdeps .                                     # install all the deps for the current directory
      cpanm -L extlib Plack                                     # install Plack and all non-core deps into extlib
      cpanm --mirror http://cpan.cpantesters.org/ DBI           # use the fast-syncing mirror
      cpanm -M https://cpan.metacpan.org App::perlbrew          # use only this secure mirror and its index
    
    You can also specify the default options in PERL_CPANM_OPT environment variable in the shell rc:
    
      export PERL_CPANM_OPT="--prompt --reinstall -l ~/perl --mirror http://cpan.cpantesters.org"
    
    Type `man cpanm` or `perldoc cpanm` for the more detailed explanation of the options.

