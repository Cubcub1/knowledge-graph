# 问题：
git push时报如图的问题

![](https://cdn.jsdelivr.net/gh/Cubcub1/ImageRepo/obsidian/202211231334586.png)

# fix
[类似问题](https://github.com/vernesong/OpenClash/issues/2074)

随后修改ssh git的端口为443，具体步骤如下：

1. vim ～/.ssh/confg  
```
Host github.com
Hostname ssh.github.com
Port 443
User git
```

2. 测试
```shell
$ ssh -T git@github.com
> Hi USERNAME! You've successfully authenticated, but GitHub does not
> provide shell access.
```

[git ssh 问题排查文档](https://docs.github.com/cn/authentication/troubleshooting-ssh/using-ssh-over-the-https-port)
