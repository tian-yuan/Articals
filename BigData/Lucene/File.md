### Lucene File



* .fnm

The .fnm file contains all the field names used by documents in the associated segment.

* .tis/.tll

All terms (tuples of field name and value) in a segment are stored in the .tis file.

* .frq

Term frequencies in each document are listed in the .frq file.

* .prx

The .prx file lists the position of each term within a document. 



![image-20190719194939868](/Users/tianyuan/Library/Application Support/typora-user-images/image-20190719194939868.png)

* .fdx

The .fdx file contains simple index information, which is used to resolve document number to exact position in the .fdt file for that document’s stored fields.

* .fdt

The .fdt file contains the contents of the fields that were stored.

* .tvf

Term vectors are stored in three files. The .tvf file is the largest and stores the specific terms, sorted alphabetically, and their frequencies, plus the optional offsets and positions for the terms. 

* .tvd

The .tvd file lists which fields had term vectors for a given document and indexes byte offsets into the .tvf file so specific fields can be retrieved.

* .tvx

the .tvx file has index information, which resolves document numbers into the
byte positions in the .tvf and .tvd files.

* .nrm

The .nrm file contains normalization factors that represent the boost information gathered during indexing.



![img](http://www.jqpress.com/upfiles/201205/field.jpg)

* 域数据文件(fdt):


真正保存存储域(stored field)信息的是fdt文件，在一个段(segment)中总共有segment size篇文档，所以fdt文件中共有segment size个项，每一项保存一篇文档的域的信息对于每一篇文档，一开始是一个fieldcount，也即此文档包含的域的数目，接下来是 fieldcount 个项，每一项保存一个域的信息。

 对于每一个域，fieldnum是域号，接着是一个8位的byte，最低一位表示此域是否分词(tokenized)，倒数第二位表示此域是保存字符串数据还是二进制数据，倒数第三位表示此域是否被压缩，再接下来就是存储域的值，比如new Field("title", "lucene in action",Field.Store.Yes, …)，则此处存放的就是"lucene in action"这个字符串。

* 域索引文件(fdx)

 由域数据文件格式我们知道，每篇文档包含的域的个数，每个存储域的值都是不一样的，因而域数据文件中segment size篇文档，每篇文档占用的大小也是不一样的，那么如何在fdt中辨别每一篇文档的起始地址和终止地址呢，如何能够更快的找到第n篇文档的存储域的信息呢？就是要借助域索引文件。

域索引文件也总共有segment size个项，每篇文档都有一个项，每一项都是一个long，大小固定，每一项都是对应的文档在fdt文件中的起始地址的偏移量，这样如果我们想找到第n篇文档的存储域的信息，只要在fdx中找到第n项，然后按照取出的long作为偏移量，就可以在fdt文件中找到对应的存储域的信息。

 