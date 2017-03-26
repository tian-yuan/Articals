#jemalloc性能分析

*	测试分配相同内存的性能--单线程

```
#include <stdlib.h>
#include <stdio.h>
#include <stdint.h>
#include <sys/time.h>

int64_t NowInUsecTime() {
    struct timeval tv;
    gettimeofday(&tv, 0);
    int64_t current_time = tv.tv_sec;
    return current_time * 1000000 + tv.tv_usec;
}

void
do_something(size_t i)
{
	void * p = malloc(100);
	free(p);
}

int
main(int argc, char **argv)
{
    size_t i,j,k;
	for(k = 0; k < 10; k++) {
       	int64_t begin = NowInUsecTime();
       	for (i = 0; i < 10000; i++) {
        	for (j = 0; j < 1000; j++) {
        		do_something(j);
            }
       	}
		int64_t end = NowInUsecTime();
		printf("use time : %lld\n", end - begin);
	}
	return (0);
}
```


```
use time : 616943
use time : 609373
use time : 606981
use time : 606367
use time : 610980
use time : 702372
use time : 701877
use time : 665435
use time : 604379
use time : 634371
单位是us
```

*	测试使用jemalloc分配相同内存的性能--单线程
	
	gcc jemalloc_test.c -o ex_stats_print -ljemalloc
	
```
use time : 257144
use time : 250584
use time : 247579
use time : 248259
use time : 250576
use time : 248948
use time : 249950
use time : 251564
use time : 253156
use time : 252750
```

*	测试使用tcmalloc分配相同内存的性能--单线程
	
	gcc jemalloc_test.c -o ex_stats_print -ltcmalloc
	
```
use time : 415271
use time : 440675
use time : 421859
use time : 416378
use time : 454048
use time : 591450
use time : 506364
use time : 426325
use time : 433321
use time : 457162	
```


*	测试分配不同内存的性能--单线程

```
void
do_something(size_t i)
{
	void * p = malloc(i*100);
	free(p);
}
```
```
use time : 770880
use time : 769960
use time : 760601
use time : 764168
use time : 783218
use time : 778257
use time : 759824
use time : 757523
use time : 762915
use time : 762817
```

*	测试使用jemalloc分配不同内存的性能--单线程

```
use time : 4120897
use time : 3738390
use time : 3711186
use time : 3721199
use time : 3729040
use time : 3945766
use time : 4031800
use time : 4009522
use time : 3756754
use time : 3758226
```

*	测试使用tcmalloc分配不同内存的性能--单线程

```
use time : 423994
use time : 429568
use time : 594751
use time : 566355
use time : 478466
use time : 454334
use time : 483270
use time : 473061
use time : 439809
use time : 429963
```


>从上面测试可以看出，分配相同的内存性能会有大幅提升，但是分配不同大小的内存，jemalloc的性能下降很多，相比来说 tcmalloc 的性能都比较优秀

```
void
do_something(size_t i)
{
       	i = (i % 2 == 0 ? 0 : 1);
        void * p = malloc(100 + i);
       	free(p);
}
```

```
liuzebodeMacBook-Pro-2:TestCode liuzebo$ ./ex_stats_print
use time : 773278
use time : 774564
use time : 762454
use time : 771430
use time : 766882
use time : 783539
use time : 773958
use time : 792290
use time : 781083
use time : 774308
liuzebodeMacBook-Pro-2:TestCode liuzebo$ otool -L ./ex_stats_print
./ex_stats_print:
       	/usr/local/lib/libjemalloc.2.dylib (compatibility version 0.0.0, current version 0.0.0)
       	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1226.10.1)
liuzebodeMacBook-Pro-2:TestCode liuzebo$ gcc ex_stats_print.c -o ex_stats_print -ltcmalloc
liuzebodeMacBook-Pro-2:TestCode liuzebo$ ./ex_stats_print
use time : 414860
use time : 410769
use time : 425743
use time : 408366
use time : 404848
use time : 415181
use time : 408616
use time : 408856
use time : 415079
use time : 405658
liuzebodeMacBook-Pro-2:TestCode liuzebo$ gcc ex_stats_print.c -o ex_stats_print
liuzebodeMacBook-Pro-2:TestCode liuzebo$ otool -L ./ex_stats_print
./ex_stats_print:
       	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1226.10.1)
liuzebodeMacBook-Pro-2:TestCode liuzebo$ ./ex_stats_print
use time : 623317
use time : 612710
use time : 611884
use time : 605058
use time : 621751
use time : 609508
use time : 620272
use time : 615481
use time : 608515
use time : 618781
```

从这次测试可以看出jemalloc分配内存有一点点不一样，性能就下降非常多
