可以通过修改编辑.profile或.bash_profile文件进行终端个性化设置。全局的设置一般是在/ect/profile里，但不建议修改此文件。

一般修改完.bash_profile文件后，会在下次进入终端时显示，如果要立即生效，则执行命令`source .bash_profile`即可。

下面是设置样例：

```bash
export CLICOLOR=1
export LSCOLORS=ExFxBxDxCxegedabagbdad

#enables colorfor iTerm
exportTERM=xterm-color
#设置命令提示符
#PS1="[\u@\h \W]\$"
PS1="\$"

#设置指令别名
alias ll='ls -l'

#配置当前用户下的全局命令查找目录PATH
PATH=$PATH:$HOME/bin
export PATH
```


