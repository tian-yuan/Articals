https://zh.wikipedia.org/wiki/Burrows-Wheeler%E5%8F%98%E6%8D%A2



```
END = '\1'  #必须不与原字符串中任何字符相同

def bwt(s):
    """对字符串进行Burrows-Wheeler变换"""
    s = s + END
    #创建所有循环字符串，每次左移一位，获取所有左移后的字符串，对所以这些字符串做字典排序，然后获取排序字符串最后一个字符构建 L 字符串
    table = [s[i:] + s[:i] for i in range(len(s))]
    #获取排序后的结果
    table_sorted = table[:]
    table_sorted.sort()
    #取排序后结果的最后一列作为结果字符串
    return ''.join([row[-1] for row in table_sorted])

def ibwt(r):
    table = [''] * len(r)
    for _ in r:
        # 创建所有的转换字符串
        table = sorted([r[m] + table[m] for m in range(len(r))])
    s = [row for row in table if row.endswith(END)][0]
    return s.rstrip(END)
```

