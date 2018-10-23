#### HBASE Command

* create

```
create 't1', {NAME => 'f1'}, {NAME => 'f2'}
```

* put

```
put 't1', 'msgId-1', 'f1', '1111'
```

* get

```
hbase(main):016:0> get 't1', 'msgId-1'
COLUMN                              CELL
 f1:                                timestamp=1528344272720, value=222233333
 f2:                                timestamp=1528344330671, value=device info
1 row(s) in 0.0280 seconds
```

* append

Append只有在 Put 的同时还需要 Get 的情况下才需要使用，比如你有个数据项是 list：a,b,c

如果现在需要在这个数据项后面追加 ",d"，如果不用 Append，就需要先 Get，然后组合成 "a,b,c,d" 后再 Put，需要两次交互，同时还会产生竞争状态。而 Append 命令只需要一次操作即可完成。

```
hbase(main):013:0> append 't1', 'msgId-1', 'f1:', '33333'
0 row(s) in 0.0630 seconds

hbase(main):017:0> get 't1', 'msgId-1'
COLUMN                     CELL
 f1:                       timestamp=1528344272720, value=222233333
 f2:                       timestamp=1528344330671, value=device info
1 row(s) in 0.0540 seconds
```

