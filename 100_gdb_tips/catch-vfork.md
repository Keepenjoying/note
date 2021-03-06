# 為vfork調用設置catchpoint
## 例子
	#include <stdio.h>
	#include <stdlib.h>
	#include <sys/types.h>
	#include <unistd.h>
	
	int main(void) {
	    pid_t pid;
	
	    pid = vfork();
	    if (pid < 0)
	    {
	        exit(1);
	    }
	    else if (pid > 0)
	    {
	        exit(0);
	    }
	    printf("hello world\n");
	    return 0;
	}



## 技巧
使用gdb調試程序時，可以用“`catch vfork`”命令為`vfork`調用設置`catchpoint`，以上面程序為例：  

	(gdb) catch vfork
	Catchpoint 1 (vfork)
	(gdb) r
	Starting program: /home/nan/a
	
	Catchpoint 1 (vforked process 27312), 0x00000034e42acfc4 in vfork ()
	   from /lib64/libc.so.6
	(gdb) bt
	#0  0x00000034e42acfc4 in vfork () from /lib64/libc.so.6
	#1  0x0000000000400561 in main () at a.c:9

可以看到當`vfork`調用發生後，gdb會暫停程序的運行。  
注意：目前只有HP-UX和GNU/Linux支持這個功能。  
參見[gdb手冊](https://sourceware.org/gdb/onlinedocs/gdb/Set-Catchpoints.html).

## 貢獻者

nanxiao
