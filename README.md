### CentOS 搭建SVN并用钩子自动实现同步到web目录教程

##### 安装

```
yum install subversion
```

##### 检查是否安装成功

```
svnserve –version
```

##### 创建仓库目录

```
mkdir –p /home/svn/test
```

##### 创建项目

```
svnadmin create /home/svn/test
```

##### 检查是否创建成功

```
cd /home/svn/test   +  ll
```

![](http://segmentfault.com/img/bVqVbh)

##### 进入conf目录更改配置文件

###### authz:用户权限配置

```
### This file is an example authorization file for svnserve.
### Its format is identical to that of mod_authz_svn authorization
### files.
### As shown below each section defines authorizations for the path and
### (optional) repository specified by the section name.
### The authorizations follow. An authorization line can refer to:
###  - a single user,
###  - a group of users defined in a special [groups] section,
###  - an alias defined in a special [aliases] section,
###  - all authenticated users, using the '$authenticated' token,
###  - only anonymous users, using the '$anonymous' token,
###  - anyone, using the '*' wildcard.
###
### A match can be inverted by prefixing the rule with '~'. Rules can
### grant read ('r') access, read-write ('rw') access, or no access
### ('').

[aliases]
# joe = /C=XZ/ST=Dessert/L=Snake City/O=Snake Oil, Ltd./OU=Research Institute/CN=Joe Average

[groups]
# harry_and_sally = harry,sally
# harry_sally_and_joe = harry,sally,&joe
[/]
testusername = rw

# [/foo/bar]
# harry = rw
# &joe = r
# * =

# [repository:/baz/fuz]
# @harry_and_sally = rw
# * = r
```

###### passwd:用户密码

```
### This file is an example password file for svnserve.
### Its format is similar to that of svnserve.conf. As shown in the
### example below it contains one section labelled [users].
### The name and password for each user follow, one account per line.

[users]
# harry = harryssecret
# sally = sallyssecret
testusername =123456
```

###### svnserve.conf

```
### This file controls the configuration of the svnserve daemon, if you
### use it to allow access to this repository.  (If you only allow
### access through http: and/or file: URLs, then this file is
### irrelevant.)

### Visit http://subversion.tigris.org/ for more information.

[general]
### These options control access to the repository for unauthenticated
### and authenticated users.  Valid values are "write", "read",
### and "none".  The sample settings below are the defaults.
anon-access = read
auth-access = write
### The password-db option controls the location of the password
### database file.  Unless you specify a path starting with a /,
### the file's location is relative to the directory containing
### this configuration file.
### If SASL is enabled (see below), this file will NOT be used.
### Uncomment the line below to use the default password file.
password-db = passwd
### The authz-db option controls the location of the authorization
### rules for path-based access control.  Unless you specify a path
### starting with a /, the file's location is relative to the the
### directory containing this file.  If you don't specify an
### authz-db, no path-based access control is done.
### Uncomment the line below to use the default authorization file.
authz-db = authz
### This option specifies the authentication realm of the repository.
### If two repositories have the same authentication realm, they should
### have the same password database, and vice versa.  The default realm
### is repository's uuid.
realm = /home/svn/test

[sasl]
### This option specifies whether you want to use the Cyrus SASL
### library for authentication. Default is false.
### This section will be ignored if svnserve is not built with Cyrus
### SASL support; to check, run 'svnserve --version' and look for a line
### reading 'Cyrus SASL authentication is available.'
# use-sasl = true
### These options specify the desired strength of the security layer
### that you want SASL to provide. 0 means no encryption, 1 means
### integrity-checking only, values larger than 1 are correlated
### to the effective key length for encryption (e.g. 128 means 128-bit
```

##### 启动SVN服务

```
svnserve -d -r /home/svn     --注意：不是svnserve -d -r /home/svn/test
```

##### 在本机安装TortoiseSVN

```
输入svn://你的ip地址/test

如果弹出输入账号密码的对话框，表示上述设置正确。
```

##### Svn commit后自动同步到网站根目录的方法

**流程如下：本地文件commit到仓库->仓库同步到根目录，可以把仓库理解为一个中转站。进入到项目下的hooks文件夹**

```
1、cd /home/svn/test/hooks
2、把post-commit.tmpl的扩展名去掉，并赋予777权限
3、vim post-commit写入如下内容
#!/bin/sh

export LANG=zh_CN.UTF-8
svn co svn://你的ip/test /home/wwwroot/test/web

# POST-COMMIT HOOK

# The post-commit hook is invoked after a commit.  Subversion runs
# this hook by invoking a program (script, executable, binary, etc.)
# named 'post-commit' (for which this file is a template) with the
# following ordered arguments:
#
#   [1] REPOS-PATH   (the path to this repository)
#   [2] REV          (the number of the revision just committed)
#
# The default working directory for the invocation is undefined, so
# the program should set one explicitly if it cares.
#
# Because the commit has already completed and cannot be undone,
# the exit code of the hook program is ignored.  The hook program
# can use the 'svnlook' utility to help it examine the
# newly-committed tree.
#
# On a Unix system, the normal procedure is to have 'post-commit'
# invoke other programs to do the real work, though it may do the
# work itself too.
#
# Note that 'post-commit' must be executable by the user(s) who will
# invoke it (typically the user httpd runs as), and that user must
# have filesystem-level permission to access the repository.
#
# On a Windows system, you should name the hook program
# 'post-commit.bat' or 'post-commit.exe',
# but the basic idea is the same.
#
# The hook program typically does not inherit the environment of
# its parent process.  For example, a common problem is for the
# PATH environment variable to not be set to its usual value, so
# that subprograms fail to launch unless invoked via absolute path.
# If you're having unexpected problems with a hook program, the
# culprit may be unusual (or missing) environment variables.
#
# Here is an example hook script, for a Unix /bin/sh interpreter.
# For more examples and pre-written hooks, see those in
```

##### 多版本控制

按照同样的方法在仓库目录中新建项目，操作相同