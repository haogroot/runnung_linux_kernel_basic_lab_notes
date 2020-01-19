# lab1 寫一個簡單的 character device driver


## Question 
[x] why device driver can be created under /proc file system?

網路上找到的資料幾乎都以 `proc_create` 為範本，但是 lab1 並不是使用這個方法，卻也在 /proc 下生成node, ~~ 原本懷疑是透過 insmod ，但每個 kernel module 都必須使用 insmod 來載入此 module ~~

[/proc/devices/](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/4/html/Reference_Guide/s2-proc-devices.html) ：這篇提到在 /proc/devices 路徑下會顯示出被載入的 block devices 或 character devices
<linux_kernel>/Documentation/filesystems/proc.txt: 同樣寫到在 /proc/devices 下，會顯示出 Available devices (block or character)

> 因此可以說 透過 insmod 將 module 載入到 kernel 後，device driver 就會以 kernel info 的情況顯示在 /proc/devices 下。


# lab6 using blocking 

## Question
[o] 我利用 cat 指令 開始兩個在 read_queue 等待的 process，接著透過 echo 寫資料到 kfifo 中，這兩個 read process 會怎麼動作呢？

 若是一次寫的資料量小於 fifo 最大深度，兩個 read process 會被輪流喚醒來讀取 data。
 若是一次寫的資料量大於 fifo 最大深度，兩個 read process 也會被輪流喚醒讀走不同段的資料。
 
 [ ] 當 read process 讀完一段資料後，又會再觸發一次 read operation，使得它又再度進入睡眠狀態，為什麼會再度觸發呢？
 
>/mnt # echo "Today is so cold." > /dev/my_demo_dev 
>[13833.466624] demodrv_open: major=10, minor=58
>[13833.467806] demodrv_write: pid=773, actual_write =18, ppos=0, ret=0
>[13833.468264] demodrv_read, pid=781, actual_readed=18, pos=0
>Today is so cold.
>[13833.468870] demodrv_read: pid=781, going to sleep
>
 
 書中寫到 write process 寫入資料後馬上喚醒了 read process，read process 把剛才寫入的數據讀到 user space，此時 fifo 又空了，read process 就進入睡眠狀態。但為什麼 read process 在讀空 fifo 後沒有直接結束呢？ 相較之下，write process 會直接結束。

 
# lab8 為什麼不能喚醒讀寫進程


## Question

當我利用lab7 test 程式來執行 open 與 poll，並寫 data 到 device 裡，會得到以下錯誤

>/mnt # echo "I am study linux kernel right now." > /dev/mydemo0
>[  740.254120] demodrv_open: major=252, minor=0, device=my_demo_dev0
>[  740.261034] wait up read queue, private_data_0
>[  740.261376] demodrv_write:my_demo_dev0 pid=773, actual_write =35, ppos=0, ret=0
>[  740.263077] Unable to handle kernel paging request at virtual address c789f6c0
>[  740.263628] pgd = c4dd0000
>[  740.263929] [c789f6c0] *pgd=00000000
>[  740.265584] Internal error: Oops: 5 [#1] SMP ARM
>[  740.266159] Modules linked in: mydemo_error(O)
>[  740.267096] CPU: 0 PID: 773 Comm: sh Tainted: G           O    4.0.0+ #1
>[  740.267404] Hardware name: ARM-Versatile Express
>[  740.267973] task: c50c5d80 ti: c478c000 task.ti: c478c000
>[  740.268987] PC is at kfree+0x178/0x704
>[  740.269314] LR is at __kfifo_free+0x1c/0x64

> 推測可能是使用記憶體上的不當？！


 