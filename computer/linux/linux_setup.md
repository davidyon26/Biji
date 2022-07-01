# Linux Setup
The following tools should be installed 

## install zsh
```zsh
  sudo apt-get install zsh
```
## set proxy
```zsh
  export http_proxy="http://pt-proxy-prd02.infinera.com:3128"
  export https_proxy="http://pt-proxy-prd02.infinera.com:3128"
```
  
## install ohmyzsh
```zsh
  sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## set vim environment

* install dep lib for vim YCM
```zsh    
  apt install build-essential cmake vim-nox python3-dev
  apt install mono-complete golang nodejs default-jdk npm
```

* create a YCM configuration file .ycm_extra_conf.py
```python
  def Settings( **kwargs ):
        return {
                'interpreter_path': '../../../pyvenv/g30p37/bin/python'
        }
```
* install 'plint' in python virtual environment for syntax check used by ALE

* install tag
```zsh
  sudo apt-get install universal-ctags
  sudo apt-get install ack
```

## install python3.7.13 from source
It is installed on ubuntu 20.04 (failed on ubuntu 22.04
due to openssl verson is 3.0.x on 22.04 and 3.7.13 not suit for it
```zsh
  wget https://www.python.org/ftp/python/3.7.13/Python-3.7.13.tar.xz
  tar -xf Python-3.7.13.tar.xz
  sudo mv Python3.7.{version} /opt/
```
* install python dependencies
uncomment or add deb-src corresponding with deb in source.list of apt
```zsh
  sudo apt-get update
  sudo apt-get build-dep python3
  sudo apt-get install pkg-config
  sudo apt-get install build-essential gdb lcov pkg-config \
        libbz2-dev libffi-dev libgdbm-dev libgdbm-compat-dev liblzma-dev \
        libncurses5-dev libreadline6-dev libsqlite3-dev libssl-dev \
        lzma lzma-dev tk-dev uuid-dev zlib1g-dev
  sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libreadline-dev libffi-dev curl libbz2-dev -y
  cd /opt/Python3.7.13/
  ./configure --enable-optimizations --enable-shared
```
* the install script with do test and verify no failure
```zsh
  make -j 6   
  sudo make altinstall
  sudo ldconfig /opt/Python3.7.13
```
* verify build successfully
```zsh
  python3.7 --version
```
## tools can be used by zsh or vim

- show file content
  - [bat](https://github.com/sharkdp/bat)
    
    like cat with syntax highlight
  
- search file by name
  - [fzf](https://github.com/junegunn/fzf.vim)

- search file by content
  - [ack](https://beyondgrep.com/)
    ```zsh
    sudo apt-get install ack
    ```  
  - [ag](https://geoff.greer.fm/ag/)
    ```zsh
    sudo apt-get install silversearcher-ag
    ```
  - [ripgrep](https://github.com/BurntSushi/ripgrep)
    ```zsh
    sudo apt-get install ripgrep
    ```
  - [git-grep](https://git-scm.com/docs/git-grep)
  - [Gnu grep](https://www.gnu.org/software/grep/)
  
  There is a doc on [feature comparison](https://beyondgrep.com/feature-comparison/) of them
