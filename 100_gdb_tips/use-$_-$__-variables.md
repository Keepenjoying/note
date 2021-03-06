# 使用“$\_”和“$__”變量
## 例子
	#include <stdio.h>

	int main(void)
	{
	        int i = 0;
	        char a[100];
	
	        for (i = 0; i < sizeof(a); i++)
	        {
	                a[i] = i;
	        }
	
	        return 0;
	}

## 技巧
"`x`"命令會把最後檢查的內存地址值存在“`$_`”這個“convenience variable”中，並且會把這個地址中的內容放在“`$__`”這個“convenience variable”，以上面程序為例:
	
	(gdb) b a.c:13
	Breakpoint 1 at 0x4004a0: file a.c, line 13.
	(gdb) r
	Starting program: /data2/home/nanxiao/a
	
	Breakpoint 1, main () at a.c:13
	13              return 0;
	(gdb) x/16xb a
	0x7fffffffe4a0: 0x00    0x01    0x02    0x03    0x04    0x05    0x06    0x07
	0x7fffffffe4a8: 0x08    0x09    0x0a    0x0b    0x0c    0x0d    0x0e    0x0f
	(gdb) p $_
	$1 = (int8_t *) 0x7fffffffe4af
	(gdb) p $__
	$2 = 15


可以看到“`$_`”值為`0x7fffffffe4af`，正好是"`x`"命令檢查的最後的內存地址。而“`$__`”值為`15`。  
另外要注意有些命令（像“`info line`”和“`info breakpoint`”）會提供一個默認的地址給"`x`"命令檢查，而這些命令也會把“`$_`”的值變為那個默認地址值：

	(gdb) p $_
	$5 = (int8_t *) 0x7fffffffe4af
	(gdb) info breakpoint
	Num     Type           Disp Enb Address            What
	1       breakpoint     keep y   0x00000000004004a0 in main at a.c:13
	        breakpoint already hit 1 time
	(gdb) p $_
	$6 = (void *) 0x4004a0 <main+44>


可以看到使用“`info breakpoint`”命令後，“`$_`”值變為`0x4004a0`。  
參見[gdb手冊](https://sourceware.org/gdb/onlinedocs/gdb/Convenience-Vars.html).

## 貢獻者

nanxiao
