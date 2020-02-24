# GDB 简要使用表 {#gdb-简要使用表}

| **命令** | **解释** | **示例** |
| :--- | :--- | :--- |
| file &lt;文件名&gt; | 加载被调试的可执行程序文件。 因为一般都在被调试程序所在目录下执行GDB，因而文本名不需要带路径。 | \(gdb\) file gdb-sample |
| r | Run的简写，运行被调试的程序。 如果此前没有下过断点，则执行完整个程序；如果有断点，则程序暂停在第一个可用断点处。 | \(gdb\) r |
| c | Continue的简写，继续执行被调试程序，直至下一个断点或程序结束。 | \(gdb\) c |
| b &lt;行号&gt; b &lt;函数名称&gt; b_&lt;函数名称&gt; b_&lt;代码地址&gt;d \[编号\] | b: Breakpoint的简写，设置断点。两可以使用“行号”“函数名称”“执行地址”等方式指定断点位置。 其中在函数名称前面加“\*”符号表示将断点设置在“由编译器生成的prolog代码处”。如果不了解汇编，可以不予理会此用法。d: Delete breakpoint的简写，删除指定编号的某个断点，或删除所有断点。断点编号从1开始递增。 | \(gdb\) b 8 \(gdb\) b main \(gdb\) b_main \(gdb\) b_0x804835c\(gdb\) d |
| s, n | s: 执行一行源程序代码，如果此行代码中有函数调用，则进入该函数； n: 执行一行源程序代码，此行代码中的函数调用也一并执行。s 相当于其它调试器中的“Step Into \(单步跟踪进入\)”； n 相当于其它调试器中的“Step Over \(单步跟踪\)”。这两个命令必须在有源代码调试信息的情况下才可以使用（GCC编译时使用“-g”参数）。 | \(gdb\) s \(gdb\) n |
| si, ni | si命令类似于s命令，ni命令类似于n命令。所不同的是，这两个命令（si/ni）所针对的是汇编指令，而s/n针对的是源代码。 | \(gdb\) si \(gdb\) ni |
| p &lt;变量名称&gt; | Print的简写，显示指定变量（临时变量或全局变量）的值。 | \(gdb\) p i \(gdb\) p nGlobalVar |
| display ...undisplay &lt;编号&gt; | display，设置程序中断后欲显示的数据及其格式。 例如，如果希望每次程序中断后可以看到即将被执行的下一条汇编指令，可以使用命令 “display /i $pc” 其中 $pc 代表当前汇编指令，/i 表示以十六进行显示。当需要关心汇编代码时，此命令相当有用。undispaly，取消先前的display设置，编号从1开始递增。 | \(gdb\) display /i $pc\(gdb\) undisplay 1 |
| i | Info的简写，用于显示各类信息，详情请查阅“help i”。 | \(gdb\) i r |
| q | Quit的简写，退出GDB调试环境。 | \(gdb\) q |
| x /&lt;地址&gt; | examine的x 来查看内存地址中的值。n表示要显示的内存单元的个数f表示显示方式, 可取如下值x 按十六进制格式显示变量。d 按十进制格式显示变量。u 按十进制格式显示无符号整型。o 按八进制格式显示变量。t 按二进制格式显示变量。a 按十六进制格式显示变量。i 指令地址格式c 按字符格式显示变量。f 按浮点数格式显示变量。u表示一个地址单元的长度b表示单字节，h表示双字节，w表示四字节，g表示八字节 | \(gdb\) /sw $eax |
| help \[命令名称\] | GDB帮助命令，提供对GDB名种命令的解释说明。 如果指定了“命令名称”参数，则显示该命令的详细说明；如果没有指定参数，则分类显示所有GDB命令，供用户进一步浏览和查询。 | \(gdb\) help display |


