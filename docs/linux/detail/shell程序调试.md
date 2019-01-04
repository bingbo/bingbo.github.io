set命令可以辅助脚本调试。

* set -n    读命令但并不执行
* set -v    显示读取的所有行
* set -x    显示所有命令及其参数

> set - 为打开相应的命令，set + 为关闭调试命令。一般在脚本开始打开调试set -x，在脚本尾部关闭调试set +x。

```bash
$./test.txt aa bb cc
+ echo hxB
hxB
+ echo '\n'
\n
+ echo aa bb cc
aa bb cc
+ set +x
```
