#### Using a jump list

Like a web browser, you can go back, then forward:

- Press Ctrl-O to jump back to the previous (older) location.
- Press Ctrl-I (same as Tab) to jump forward to the next (newer) location.



You can go back to the last buffer using `:b#`.

If you just opened a file, then it will bring you just back to the directory browsing.

**Update**: Since this answer happened to be accept as the correct answer and is thus on the top, I'd like to summarize a bit the answers, including the one by @romainl that imho is the correct one.

- `:Rex[plore]`: Return to Explorer (by @romainl) [vimdoc.sourceforge](http://vimdoc.sourceforge.net/htmldoc/pi_netrw.html#:Rexplore)
- `:Ex`: opens the Explorer, but mustn't necessarily be the same (by @drug_user841417). See [vim.wikia](http://vim.wikia.com/wiki/File_explorer)
- `:b#`: goes back to the "previously edited buffers". See [vim.wikia](http://vim.wikia.com/wiki/Easier_buffer_switching#Switching_to_the_previously_edited_buffer)
- `Ctrl`-`O`: jump back to the previous (older) location, not necessarily a buffer (by @Peyman). See [vim.wikia](http://vim.wikia.com/wiki/Jumping_to_previously_visited_locations)
- Ctrl + 6: on mac



From within Vim, new files are created like existing files are edited, via commands like `:edit filename` or `:split filename`. To persist them to disk, you need to (optionally type in contents and) persist them via `:write`.

Like a command prompt, Vim has a notion of *current directory* (`:pwd` lists it). All file paths are relative to it. You don't need to duplicate the path to your current file, there are some nice shortcuts for them: `%` refers to the current file, `:h` is a modifier for its directory, minus the file name (cp. `:help filename-modifiers`). So,

```
:e %:h/filename
:w
```

will create a new file named `filename` in the same directory as the currently open file, and write it.

Alternatively, some people like Vim to always change to the current file's directory. This can be configured by placing

```
:set autochdir
```

into your `~/.vimrc` file (which is read on Vim startup). Then, above becomes simply

```
:e filename
:w
```

Finally, Vim has a great built-in `:help`. Learn to navigate and search it!



I would shell out for either of these commands. In command mode:

```
:! touch new-file.txt
:! mkdir new-directory
```

A great plugin for these types of actions is [vim-eunuch](https://github.com/tpope/vim-eunuch), which gives you a lot of sugar for the UNIX shell commands. Here's the latter example using vim-eunuch:

```
:Mdkir new-directory
```

#### install ctag

mac自带的ctags程序不是exuberant ctags， 所以使用时会出现问题，所以要重新安装一个；

```
brew install ctags-exuberant 
```

安装完， which ctags

如果是/usr/bin/ctags,系统默认先看到我们安装的ctags

打开~/根目录下的.profile，如果你也没发现有这个文件，没关系，创建一个！

然后在里面添加：export PATH="/usr/local/bin:/usr/local/sbin:$PATH"

再到终端执行：source ~/.profile

然后再看看which ctags，如无意外，应该是/usr/local/bin/ctags

最后在.vimrc配置文件添加：

```
let Tlist_Ctags_Cmd="/usr/local/bin/ctags"
let Tlist_Show_One_File=1
let Tlist_Exit_OnlyWindow=1
let Tlist_Use_Right_Window=1
```

2、使用ctags编译项目tags文件

终端cd 项目目录，然后执行：

ctags -R

你会发现目录中多了一个tags的文件，这个就是vim里面taglist会寻找的文件！

在vim中对准某个对象调用的方法按control + ] 看看能否调到那个方法的定义！？

control + t  返回原方法

