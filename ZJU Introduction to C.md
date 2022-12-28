# First C program

- several example:

```c
#include <stdio.h>

int main()
{
    printf("Hello World!\n");
    return 0;
}
```



```c
#include <stdio.h>

int main()
{
    printf("23+43=%d\n", 23+43);

    return 0;
}
```



- gcc: 
  - GNU compiler collection
  - most widely-used compiler for c
  - use gcc in window:
    - cygwin: provide a interface for runing unix program in windows
      - `unix lib` + `.c` -----compiler--------> `program runnable in unix`  -----cygwin------> running in win
    - MinGW(minimal GNU for windows)
      - .c  + winlib  --------compiler---------> .exe
      - stopped updating
    - other choices: TDM-GCC





# Calculation

## 变量

- 初始化: 在C语言中, 变量若在声明时没有赋值, 之后直接调用时会产生任意可能的结果. 因为此时变量只是分配到某一内存地址, 其内可能有任意内容, 而我们没有覆写该地址中的值.
- 定义变量
  - c99允许在程序的任何地方定义变量 (和绝大多数编程语言一样)
  - ANSI C只允许在程序开头定义变量

- 例:

  ```c
  #include <stdio.h>
  
  int main()
  {
      int price = 0;
  
      printf("请输入金额（元）：");
      scanf("%d", &price);		
      // scanf() 读取命令行输入; 用%d注明要输入的数据
      // &: 指针; 不使用可能会报错
      // 若输入值不是整数, 如字符串, 实际的输入值为0 (若输入小数?)
      										
      int change = 100 - price;
  
      printf("找您%d元。\n", change);
  
      return 0;
  }
  ```

  

## scanf

- scanf要求输入的值和第一个参数的格式完全一致;
  - 出现在scanf内的字符串是要求你一定要输入的东西



