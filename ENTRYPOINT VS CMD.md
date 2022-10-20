## exec模式和shell模式 
> exec 模式，docker容器中的任务进程就是容器内的1号进程
> shell 模式，docker 会以/bin/sh -c "command"的当时执行任务进程。所以容器中的1号进程不是任务进程而是bash进程

**推荐使用exec模式，这样任务进程是容器的1号进程，可以使用docker的stop优雅关闭**

### CMD指令
> 为容器提供默认的执行命令

#### CMD三种运用
- **CMD ["param1","param2"]**  为ENTRYPOINT 提供默认的参数
- **CMD ["executable","param1","param2"]**    // 这是 exec 模式的写法。
- **CMD command param1 param2**                  // 这是 shell 模式的写法。

>注意命令行参数可以覆盖 CMD 指令的设置，但是只能是重写，却不能给 CMD 中的命令通过命令行传递参数。

### ENTYRPOINT指令
> 为容器提供默认的执行命令

#### ENTRYPOINT的用法
- **ENTRYPOINT ["executable", "param1", "param2"]**   // 这是 exec 模式的写法 。
- **ENTRYPOINT command param1 param2**                   // 这是 shell 模式的写法。

> exec模式，docker run命令行上指定的参数会添加到ENTRYPOINT指定命令的参数列表中
> shell模式，会完全忽略命令行参数

**ENTRYPOINT指令的覆盖，需要显式的指定 --entrypoint参数**

#### Dockerfile 中至少要有一个CMD/ENTRYPOINT

#### 同时使用CMD和ENTRYPOINT
 - 如果 ENTRYPOINT 使用了 shell 模式，CMD 指令会被忽略。  
 - 如果 ENTRYPOINT 使用了 exec 模式，CMD 指定的内容被追加为 ENTRYPOINT 指定命令的参数。 
 - 如果 ENTRYPOINT 使用了 exec 模式，CMD 也应该使用 exec 模式。
![](https://cdn.jsdelivr.net/gh/Cubcub1/ImageRepo/obsidian/202210141601767.png)
