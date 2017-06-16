# 包管理命令
## apt 工具
基于命令行的包管理工具，可以看作dpkg的前端; 同时它也可以看作是aptitute和synaptic的后端
### apt
  ```
  apt [-h] [-o=_config_string_] [-c=_config_file_] [-t=_target_release] [-a=architecture]
  {list | search | show | update | install pkg [{=pkg_version_number | /targe_release}] ... |
  remove pag... | upgrade | full-upgrade | edit-sources |
  {-v | --version} |
  {-h | --help}}
  ```
  * apt list
    ```
    apt list --installed
    apt list --upgradable
    apt list --all-versions  
    apt list package-name
    ```
### apt-cache
  * 用法
  ```
   apt-cache [-agipns] [-o=config_string] [-c=config_file] {gencaches |
           showpkg pkg...  | showsrc pkg...  | stats | dump | dumpavail
           | unmet | search regex...  |
           show pkg [{=pkg_version_number | /target_release}]...  |
           depends pkg [{=pkg_version_number | /target_release}]...  |
           rdepends pkg [{=pkg_version_number | /target_release}]...  |
           pkgnames [prefix]  |
           dotty pkg [{=pkg_version_number | /target_release}]...  |
           xvcg pkg [{=pkg_version_number | /target_release}]...  |
           policy [pkg...]  | madison pkg...  | {-v | --version} |
           {-h | --help}}
  ```
  * apt-cache serch \[option\] _regex_ : 基于正则表达式查询软件包
  ```
  --name-only : 只匹配软件包的名称。如果没有该选项，匹配包的名称和包的描述
  --full ：显示匹配包的记录信息
  
    apt-cache search --name-only ^apt #显示名以apt开头的软件包
  ```
  * apt-cache pkgnames [_prefix_] : 显示APT知道的包的名称
  ```
    apt-cache pkgnames apt  #显示名以apt开头的软件包
  ```
### apt-get
  * 用法
  ```
  apt-get [-asqdyfmubV] [-o=config_string] [-c=config_file]
               [-t=target_release] [-a=architecture] {update | upgrade |
               dselect-upgrade | dist-upgrade |
               install pkg [{=pkg_version_number | /target_release}]...  |
               remove pkg...  | purge pkg...  |
               source pkg [{=pkg_version_number | /target_release}]...  |
               build-dep pkg [{=pkg_version_number | /target_release}]...  |
               download pkg [{=pkg_version_number | /target_release}]...  |
               check | clean | autoclean | autoremove | {-v | --version} |
               {-h | --help}}
  ```
  * apt-get install _package-name_
    安装_package-name_软件包 
  * apt-get clean
    删除`/var/cache/apt/archives/` 和 `/var/cache/apt/archives/partial/`目录下除了锁文件的任何文件.
    这个目录存放安装包文件，通常会随着安装文件的增多不断增多

## apt-file
基于命令行的工具，用于查询软件包中的文件，不同于`dpkg -S`和`dpkg -L`，它查询apt管理库中所有的包而不局限于已经下载安装的包
* apt-file update:  在执行查询命令以前，需要使用该命令获取软件包的内容
```
  apt-file update
```
* apt-file search _file-pattern_
```
  apt-file search /usr/bin/gpg
```
## dpkg
处理Debian包.deb的基本命令，它用于安装和分析包文件，如果有任何问题将失败。apt可以看作是dpkg的前端命令，用于更友好地处理包的安装
  * dpkg -S _filename-search-pattern_
  * dpkg-query -S _filename-search-pattern_: 从已经安装的软件包里查询包含filename-pattern文件的软件包
  ```
    dpkg -S /usr/bin/gpt    # 查找文件gpt包含在哪个软件包中
  ```
  * dpkg -L _package-name_
  * dpkg-query -L _package-name_: 查询已安装的_package-name_包中包含的文件
  ```
    dpkg -L gnupg
  ```
## debXXX
* deborphan
  * 查找不被任何包依赖的包
