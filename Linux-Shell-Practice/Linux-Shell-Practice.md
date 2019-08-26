## Linux Shell 实践



[TOC]



### 一、启动及运行条件

#### 1、脚本启动方式

- `source` 或 `.`

- 脚本赋予可执行权限：[Shebang (Unix) - Wikipedia](https://en.wikipedia.org/wiki/Shebang_(Unix))，[Shebang - 维基百科](https://zh.wikipedia.org/zh-sg/Shebang)
- 使用 Bash 命令：`$ bash '脚本文件'`



#### 2、设置执行条件

[《Bash 脚本 set 命令教程》](http://www.ruanyifeng.com/blog/2017/11/bash-set.html)

```bash
set -o errexit   # -e  运行过程中命令报错则中止脚本执行，后续语句不再执行；通过 : 允许特定语句的错误结果
set -o xtrace    # -x  打印正在执行的语句
set -o nounset   # -u  使用未初始化的变量时报错
set -o pipefail  # 管道方式执行过程中，任何一部分失败则整条命令视作失败
```



### 二、语法语句

#### 1、引号

- 单引号 `'`：原始文本，不能转义
- 双引号 `"`：可具备转义内容
- 撇号（ESC 下面的那个）：执行



#### 2、变量

- 访问方式

  - 设置

  ```bash
  A_VAR='a string'
  # 或
  export A_VAR='a string' # export 的作用：传递给子 Shell
  ```

  - 读取

  ```bash
  echo $A_VAR
  # 或
  echo ${A_VAR}
  ```

  

- 环境变量的默认值

  参考 [**Advanced Bash-Scripting Guide** - Chapter 10. Manipulating Variables - 10.2. Parameter Substitution](http://www.tldp.org/LDP/abs/html/parameter-substitution.html)

  ```bash
  # 在 $MY_VAR 没有赋值时，将 $MY_VAR 的默认值设置为 'init_val'
  MY_VAR=${MY_VAR:-'init_val'}
  ```



- 特殊环境变量
  - 参数：常用的 `$@ $# $n $?`
参考：[《Shell特殊变量》](http://c.biancheng.net/cpp/view/2739.html)
  -  `$PS1` -  命令行提示符（[Linux Shell 颜色配置](https://www.jianshu.com/p/068798c375ee)）
  -  `$IFS` - Internal Field Seprator，参考[《Shell中的IFS解惑》](https://blog.csdn.net/whuslei/article/details/7187639)




- 保存执行结果到变量

```bash
MY_RET=$(ls) # 效果与撇号（`）一致，可读性更好
```



- 密码等敏感信息的保存（macOS）

```bash
$ security find-generic-password -a "所属用户" -s "密码项名称" -w
```

- [获取脚本所在位置](https://gist.github.com/Herbert8/a4166c07b47b739001493919565c819f)



#### 3、条件判断

注意引号的使用

- `test` 与 `[` 等效（不推荐）

- `[[`（推荐）



#### 4、循环

- `for`

```bash
for (( i = 0; i < 10; i++ )); do
    # 循环体
done
```



- `forin`

```bash
for i in "$@"; do # 注意引号的使用，根据情况选用
    echo "$i";
done

for i in $(seq 5 10); do
    echo "$i";
done
```



- `while`

```bash
while [[ condition ]]; do
    # 循环体
done
```



#### 5、字符串解析

- 常用的字符串处理工具
  - `$IFS` - Internal Field Seprator，参考[《Shell中的IFS解惑》](https://blog.csdn.net/whuslei/article/details/7187639)
  
  ```bash
  # 将变量 $A_VAR 以指定分隔符放至数组
  IFS='分隔符'
  AN_ARRAY=(${A_VAR})
  ```
  
  - `awk`
  - `sed`
  - `tr`
  - `cut`

- 一些使用场景
  - 获取指定网卡 IP

    需要 `ifconfig` 支持，CentOS Min 安装 通过以下命令安装网络工具：

    ```bash
    $ sudo yum install -y net-tools
    ```

    从 `ifconfig` 的返回信息中获取 IP
    
    ```bash
    $ ifconfig ens33 | sed -nr 's/.*inet (addr:)?(([0-9]*\.){3}[0-9]*).*/\2/p'
    ```
    
  - 校正服务器时间（临时替代方案）
  
    ```bash
    $ sudo date -s "$(curl -H 'Cache-Control: no-cache' -sI baidu.com | grep '^Date:' | cut -d' ' -f3-6)Z"
    ```
  
    参考：[《Linux 时间同步》](https://www.jianshu.com/p/231880efaef7)



#### 6、函数

定义的函数可以以类似命令的方式使用

```bash
# 函数定义
my_func() {
    local val_ori=$0
    local val_a=$1
    local val_b=$2

    echo ${val_ori} ${val_a} ${val_a}
}

# 传参调用函数 并 接收返回值
ret=$(my_func aa bb)

# 使用返回值
echo ${ret}
```





### 三、运行方式



#### 1、后台任务

- `&`

  - `nohup`
  - 需要正常 `exit` 一次
  - 输出重定向：`2>&1`
    - `1`：`stdout`
    - `2`：`stderr`

  参考：[《Linux 后台执行命令：& 和 nohup》](https://www.jianshu.com/p/074f82566213)
  
  ```bash
  $ nohup your_command > myout.log 2>&1 &; exit;
  ```



- 使用 [tmux](http://louiszhai.github.io/2017/09/30/tmux/) 终端
  - 多会话、多窗口、多面板
  - 状态保持



#### 2、远程命令

- [scp](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/scp.html) - 跨机远程拷贝
- ssh - 远程执行命令

```bash
$ ssh user@server.com 'ls ~/'
```



### 四、改善脚本体验

#### 1、[Dialog](https://www.jianshu.com/p/fd2122832a1e)

一个允许在脚本中展示各种问题或消息的程序，使脚本具备 TUI 的风格呈现。

表单风格：

```bash
USER_INPUT=$(dialog --title "Add a user" \
                    --output-fd 1 \
                    --form "Please input the infomation of new user:" 12 40 4  \
                        "Username:"  1  1 "" 1  15  15  0  \
                        "Full name:" 2  1 "" 2  15  15  0  \
                        "Home Dir:"  3  1 "" 3  15  15  0  \
                        "Shell:"     4  1 "" 4  15  15  0)
echo "${USER_INPUT}"
```



多选风格：

```bash
USER_INPUT=$(dialog --backtitle 'Checklist' \
                    --output-fd 1 \
                    --checklist "Test" 20 50 10 \
                        'Memory mmm' Memory_Size 1 \
                        'Dsik ddd' Disk_Size 2)
echo "${USER_INPUT}"
```



#### 2、[fzf](https://github.com/junegunn/fzf)

命令行模糊查找器。

模糊匹配选择：

```bash
$ vim $(fzf -m --border --height 40% --preview 'cat {}')
```



### 五、周边工具

#### 1、三方 Shell

- [fish](https://fishshell.com/) - **f**riendly **i**nteractive **sh**ell，[Wiki](https://zh.wikipedia.org/wiki/Fish)，参考[《Fish shell 入门教程》](http://www.ruanyifeng.com/blog/2017/05/fish_shell.html)
  - 优点：开箱即用，非常方便
  - 缺点：与 Bash 的兼容性问题
- [zsh](https://zh.wikipedia.org/wiki/Z_shell) - Z shell
  - 优点：与 Bash 兼容
  - 缺点：配置繁琐
- 个人建议
  - 交互操作使用 fish，执行脚本使用 Bash



#### 2、常用工具

- [tmux](http://louiszhai.github.io/2017/09/30/tmux/) 终端
- [vi/vim](https://www.runoob.com/linux/linux-vim.html) 文本编辑，参见 [Vim 中文社区](https://vim-china.github.io/)
- yum 包管理

```bash
# 安装 deltarpm 支持，可以只下载变化部分
$ sudo yum install -y deltarpm

# 只收集安装包及其依赖，不执行安装操作
$ sudo yum install -y --downloadonly --downloaddir='安装包保存位置' '要安装的包'
```

- netcat 网络工具
- [SSH 端口转发](https://www.jianshu.com/p/1f050174dc62)
- [frp NAT 穿透](https://github.com/fatedier/frp/blob/master/README_zh.md)
- [sshpass](https://gist.github.com/arunoda/7790979) 自动输入密码
- [7-Zip](https://zh.wikipedia.org/wiki/7-Zip)

```bash
$ sudo yum install epel-release
$ sudo yum install p7zip
```

- [`ntpdate` / `ntpd` 服务](https://blog.csdn.net/suer0101/article/details/7868813) - 时间同步



#### 3、TUI 工具

- [Midnight Commander](https://midnight-commander.org/)
    - [《如何使用Midnight Commander，一个可视文件管理器》](https://cloud.tencent.com/developer/article/1326633)
    - [7 Excellent Console Linux File Managers (Updated 2019)](https://www.linuxlinks.com/bestconsolefilemanagers/)

- [GRV](https://github.com/rgburke/grv) - Git Repository Viewer

- [dive](https://github.com/wagoodman/dive) - 功能强大的 Docker 镜像分析工具，可以查看每层镜像的具体差异等
- [ctop](https://ctop.sh/) - Docker 容器运行时资源分析，如 CPU、内存消耗等



### 六、参考

[《Bash脚本15分钟进阶教程》](https://linux.cn/article-2921-1.html)

[Linux命令大全搜索工具](https://git.io/linux)

[Linux命令大全(手册)](https://www.linuxcool.com/)





（完）