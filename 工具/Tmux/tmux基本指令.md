#### tmux 使用的基本指令

* 创建一个新的 windows

```
tmux new -s name
在 iterm 命令行输入
```

* attach 一个 tmux 会话

```
tmux attach -t name
在 iterm 命令行输入
```

* 创建一个新的 pannel

```
ctrl + b, %
在一个 tmux session 里面同时按下 ctrl + b ，然后同时松开，再按 % 号，水平切分
```

* 创建一个上下切分 pannel

```
ctrl + b, "
```

* detach 当前 tmux 回话

```
ctrl + b, d
```



> 注意，只要在 tmux 回话中，所有的指令都是 ctrl + b 开始，除非做过自定义配置

* tmux 支持滚动

https://superuser.com/questions/209437/how-do-i-scroll-in-tmux

![image-20200102135456058](../../../../Library/Application Support/typora-user-images/image-20200102135456058.png)

* Vim switch to previous buffer from within tmux

There are 2 key mappings for editing alternate file. `CTRL-6` and `CTRL-^`

Try pressing `Ctrl+Shift+6`(which is `CTRL-^`)

经过实践，在 mac 上使用 CTRL-6 可以切换到上一个

* 创建面板

![image-20200102142222835](../../../../Library/Application Support/typora-user-images/image-20200102142222835.png)

* 选择下一个面板

![image-20200102142213990](../../../../Library/Application Support/typora-user-images/image-20200102142213990.png)

* 关闭当前面板

![image-20200102142520598](../../../../Library/Application Support/typora-user-images/image-20200102142520598.png)

#### Appendix

http://louiszhai.github.io/2017/09/30/tmux/