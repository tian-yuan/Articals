1、通道关闭时间： 
一般紧跟在往通道输入最后一个数据之后。

    jobs := make(chan int, 5)
    for i := 1; i < 4; i++ {
        jobs <- i
        fmt.Println("sent job", i)
        //      if i == 3 {
        //          close(jobs)
        //      }
    }
    close(jobs)

2、读取关闭的无缓存通道： 
读取关闭后的无缓存通道，不管通道中是否有数据，返回值都为0和false。

    done := make(chan int)
    go func() {
        done <- 1
    }()
    close(done)
    for i ：= 1; i <= 3; i++ {
        t, ok := <-done
        fmt.Println(i, ":", t, ok)
    }

运行结果： 
1：0 false 
2：0 false 
3：0 false

3、读取关闭的有缓存通道： 
读取关闭后的有缓存通道，将缓存数据读取完后，再读取返回值为0和false。

    done := make(chan int， 1)
    go func() {
        done <- 1
    }()
    close(done)
    for i ：= 1; i <= 3; i++ {
        t, ok := <-done
        fmt.Println(i, ":", t, ok)
    }

运行结果： 
1：1 true 
2：0 false 
3：0 false

4、range遍历通道： 
通道写完后，必须关闭通道，否则range遍历会出现死锁。
--------------------- 
