# gcc編譯參數-fPIC的一些問題



-fPIC 作用於編譯階段，告訴編譯器產生與位置無關代碼(Position-Independent Code)，
`則產生的代碼中，沒有絕對地址，全部使用相對地址，故而代碼可以被加載器加載到內存的任意位置，都可以正確的執行`。這正是共享庫所要求的，共享庫被加載時，在內存的位置不是固定的。

```sh
gcc -shared -fPIC -o 1.so 1.c 
```

這裡有一個-fPIC參數<br>
PIC就是position independent code<br>
PIC使.so文件的代碼段變為真正意義上的共享<br>

`如果不加-fPIC,則加載.so文件的代碼段時,代碼段引用的數據對象需要重定位,重定位會修改代碼段的內容,這就造成每個使用這個.so文件代碼段的進程在內核裡都會生成這個.so文件代碼段的copy.每個copy都不一樣,取決於這個.so文件代碼段和數據段內存映射的位置.`



`不加fPIC編譯出來的so,是要再加載時根據加載到的位置再次重定位的.(因為它裡面的代碼並不是位置無關代碼)`


如果被多個應用程序共同使用,那麼它們必須每個程序維護一份so的代碼副本了.(因為so被每個程序加載的位置都不同,顯然這些重定位後的代碼也不同,當然不能共享)
我們總是用fPIC來生成so,也從來不用fPIC來生成a.
fPIC與動態鏈接可以說基本沒有關係,libc.so一樣可以不用fPIC編譯,只是這樣的so必須要在加載到用戶程序的地址空間時重定向所有表目.


因此,不用fPIC編譯so並不總是不好.<br>
如果你滿足以下4個需求/條件:<br>
```sh
1.該庫可能需要經常更新
2.該庫需要非常高的效率(尤其是有很多全局量的使用時)
3.該庫並不很大.
4.該庫基本不需要被多個應用程序共享
```
如果用沒有加這個參數的編譯後的共享庫，也可以使用的話，可能是兩個原因：
```sh
1：gcc默認開啟-fPIC選項
2：loader使你的代碼位置無關
```
從GCC來看，shared應該是包含fPIC選項的，但似乎不是所以系統都支持，所以最好顯式加上fPIC選項。參見如下

```sh
`-shared'
     Produce a shared object which can then be linked with other
     objects to form an executable.  Not all systems support this
     option.  For predictable results, you must also specify the same
     set of options that were used to generate code (`-fpic', `-fPIC',
     or model suboptions) when you specify this option.(1)
```

-fPIC 的使用，會生成 PIC 代碼，.so 要求為 PIC，以達到動態鏈接的目的，否則，無法實現動態鏈接。


`non-PIC 與 PIC 代碼的區別主要在於 access global data, jump label 的不同。`

比如一條 access global data 的指令，<br>
non-PIC 的形勢是：ld r3, var1<br>
PIC 的形式則是：ld r3, var1-offset@GOT,意思是從 GOT 表的 index 為 var1-offset 的地方處
指示的地址處裝載一個值,即var1-offset@GOT`處的4個 byte 其實就是 var1 的地址`。這個地址只有在運行的時候才知道，是由 dynamic-loader(ld-linux.so) 填進去的。

再比如 jump label 指令<br>
non-PIC 的形勢是：jump printf ，意思是調用 printf。<br>
PIC 的形式則是：jump printf-offset@GOT,<br>
`意思是跳到 GOT 表的 index 為 printf-offset 的地方處指示的地址去執行，
這個地址處的代碼擺放在 .plt section，`


`每個外部函數對應一段這樣的代碼，其功能是呼叫dynamic-loader(ld-linux.so) 來查找函數的地址(本例中是 printf)，然後將其地址寫到 GOT 表的 index 為 printf-offset 的地方，`

同時執行這個函數。這樣，第2次呼叫 printf 的時候，就會直接跳到 printf 的地址，而不必再查找了。

`GOT 是 data section, 是一個 table, 除專用的幾個 entry，每個 entry 的內容可以再執行的時候修改；
PLT 是 text section, 是一段一段的 code，執行中不需要修改。`

每個 target 實現 PIC 的機制不同，但大同小異。比如 MIPS 沒有 .plt, 而是叫 .stub，功能和 .plt 一樣。

可見，動態鏈接執行很複雜，比靜態鏈接執行時間長;但是，極大的節省了 size，PIC 和動態鏈接技術是計算機發展史上非常重要的一個里程碑。

gcc manul上面有說
-fpic        If the GOT size for the linked executable exceeds a machine-specific maximum size, you get an error message from the linker indicating that -fpic does not work; in that case, recompile with -fPIC instead. (These maximums are 8k on the SPARC and 32k on the m68k and RS/6000. The 386 has no such limit.)

-fPIC       If supported for the target machine, emit position-independent code, suitable for dynamic linking and avoiding any limit on the size of the global offset table. This option makes a difference on the m68k, PowerPC and SPARC. Position-independent code requires special support, and therefore works only on certain machines.

關鍵在於GOT全局偏移量表裡面的跳轉項大小。
intel處理器應該是統一4字節，沒有問題。
powerpc上由於彙編碼或者機器碼的特殊要求，所以跳轉項分為短、長兩種。

`-fpic為了節約內存，在GOT裡面預留了“短”長度。`

而

`-fPIC則採用了更大的跳轉項。`
