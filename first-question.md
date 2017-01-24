『當程式跑到一半，變數的值卻不知道被誰改掉了』，這個問題一定常常困擾著C programmer，這篇介紹一個能比較快找到嫌疑者的作法，只是嫌疑者，不代表能將兇手一刀斃命，畢竟始作俑者是誰這要programmer自己去看code找答案。

> 這篇有個前情提要，當一個全域變數宣告時會放在BSS，但若是宣告時有做初始化則會放在DATA。



以下直接給一個範例程式：

```
#include <stdio.h>

int array[5];

#ifdef CORRUPT
int var;
#else
int var = 777;
#endif

int main()
{
    int idx;

    var = 10;

    printf("var = %d\n", var);

    for(idx = 0; idx < 10; idx++)
    {
        array[idx] = 0;
    }

    printf("var = %d\n", var);
}
```

當使用CORRUPT這個flag去編譯時結果會是var從**10**變成**0**

```
ubuntu:~/tests/Corrupt$ gcc -g -DCORRUPT test.c
ubuntu:~/tests/Corrupt$ ./a.out
var = 10
var = 0
```

但若是直接來，那麼var的值不會被改變

```
ubuntu:~/tests/Corrupt$ gcc -g test.c
ubuntu:~/tests/Corrupt$ ./a.out
var = 10
var = 10
```

那要如何抓到兇手呢？我們可以從var的周邊變數來找下手，透過objdump可以列出變數的定址，以下我刪除多餘的部分，讓我們專注在array和var上。

```
ubuntu:~/tests/Corrupt$ objdump -t a.out

0804a01c g     O .bss   00000014              array
0804a030 g     O .bss   00000004              var
```

以上是會corrupt的情況，我們發現，array和var都被定址在BSS上，由此我們可以知道，會影響到var的人有可能是array。這時候就回頭去看code，驗證是不是在修改array時是不是有誤動作。

若是正常情況下，array依然在BSS上，但var因為初始化，所以被定址在DATA，已經不在array的附近，所以不受array的影響。這不代表array就自由了，若是城市變大，依然會有倒楣的人在array身邊而受到牽連。

```
ubuntu:~/tests/Corrupt$ objdump -t a.out

0804a020 g     O .bss   00000014              array
0804a014 g     O .data  00000004              var
```

這招可以將嫌疑者縮小，更容易找出真正的兇手，但還是有勞programmer自行從code去解析問題。