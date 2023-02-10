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

### 初始化

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

  

  

### scanf

- scanf要求输入的值和第一个参数的格式完全一致;
  - 出现在scanf内的字符串是要求你一定要输入的东西
  
    - 例: 
  
      ```c
      int main(){
          int a = 0;
          int b = 0;
          scanf("randomstr %d %d", &a, &b);
          // 只有在console中先输入`randomstr `之后, 输入字符和空格, 才能正确给a, b赋值
          return 0;
      }
      ```
  
      

### 常量

- 定义变量时, 在其类型前方加const关键字可使其成为常量
  - `const int a = 1;`



## 浮点数

- 两整数做除法运算的结果还是整数, 小数位会被truncate
- 一个浮点数和另一个整数做运算, 结果就是浮点数
- 在scanf和printf中输入/输入浮点数
  - `scanf("%lf", &numin)`
  - `printf("%f", numout)`





## 表达式

- c中的赋值符号也是一种运算符, 也有运算产生的结果

  - `a=6`的运算结果为a被赋予的值, 即6
  - `b=a=6`: 自右向左执行, 6赋给a, a=6产生6, 6赋给b

- 递增运算符

  - a++: 表达式的值是原本a的值
  - ++a: 表达式的值是a加1之后的值

  





# 指针

## 简介

- 字符`*`表示指针，通常跟在类型关键字的后面，表示指针指向的是什么类型的值。比如，`char*`表示一个指向字符的指针，`float*`表示一个指向`float`类型的值的指针。

  - 星号`*`可以放在变量名与类型关键字之间的任何地方，下面的写法都是有效的。

  - ```c
    int   *intPtr;
    int * intPtr;
    int*  intPtr;
    ```

  - 如果同一行声明两个指针变量，那么需要写成下面这样。

  - ```c
    // 正确
    int * foo, * bar;
    
    // 错误
    int* foo, bar;
    ```

  - 一个指针指向的可能还是指针，这时就要用两个星号`**`表示。

  - ```c
    int** foo;
    ```

    

## `*` `&`运算符

- `*`这个符号除了表示指针以外，还可以作为运算符，用来取出指针变量所指向的内存地址里面的值。

- `&`运算符用来取出一个变量所在的内存地址。

- `&`运算符与`*`运算符互为逆运算

- ```c
  void increment(int* p) {	// p是 int* 类型
    // *p是int类型
    *p = *p + 1;		// 因为传入的是地址，函数体内部对该地址包含的值的操作，会影响到函数外部，所以不需要返回值
  }
  
  int x = 1;
  increment(&x);
  printf("%d\n", x); // 2
  
  // 互为逆运算
  int i = 5;
  if (i == *(&i)) // 正确
  ```

  

- 理解C的变量内存结构:

<img src="C essentials.assets/image-20230117002819355.png" alt="image-20230117002819355" style="zoom:75%;" />



## 初始化

- 声明指针变量之后，编译器会为指针变量本身分配一个内存空间，但是这个内存空间里面的值是随机的，也就是说，指针变量指向的值是随机的。这时一定不能去读写指针变量指向的地址，因为那个地址是随机地址，很可能会导致严重后果。

  - ```c
    int* p;
    *p = 1; // 错误, 因为p指向的那个地址是随机的，向这个随机地址里面写入1，会导致意想不到的结果。
    ```

    

- 正确做法是指针变量声明后，必须先让它指向一个分配好的地址，然后再进行读写，这叫做指针变量的初始化。

  - ```c
    int* p;
    int i;
    
    p = &i;
    *p = 13;
    ```

    

- 为了防止读写未初始化的指针变量，可以养成习惯，将未初始化的指针变量设为`NULL`。

  - ```c
    int* p = NULL;
    ```

  - `NULL`在 C 语言中是一个常量，表示地址为`0`的内存空间，这个地址是无法使用的，读写该地址会报错。



## 指针运算

- 指针本质上就是一个无符号整数，代表了内存地址。它可以进行运算，但是规则并不是整数运算的规则。

- （1）指针与整数值的加减运算

  - 指针与整数值的运算，表示指针的移动。

  - ```c
    short* j;
    j = (short*)0x1234;
    j = j + 1; // 0x1236
    
    /*
        j + 1表示指针向内存地址的高位移动一个单位，
        而一个单位的short类型占据两个字节的宽度，
        所以相当于向高位移动两个字节。
        同样的，j - 1得到的结果是0x1232。
        数据类型占据多少个字节，每单位就移动多少个字节。
    */
    ```

    

- （2）指针与指针的加法运算

  - 指针只能与整数值进行加减运算，两个指针进行加法是非法的。

- （3）指针与指针的减法

  - 相同类型的指针允许进行减法运算，返回它们之间的距离，即相隔多少个数据单位。
  - 这时，减法返回的值属于`ptrdiff_t`类型，这是一个带符号的整数类型别名.

- （4）指针与指针的比较运算

  - 指针之间的比较运算，比较的是各自的内存地址哪一个更大，返回值是整数`1`（true）或`0`（false）。



# 函数

## 值传递机制

- 普通类型的变量如果直接作为参数传递到函数中, 那么函数中拿到的参数实际上是原本变量的拷贝, 与原本变量具有不同的内存地址
- 因此, 有时需要传指针变量
- 注意: 函数不要返回内部变量的指针。因为当函数结束运行时，内部变量就消失了，这时指向内部变量`i`的内存地址就是无效的，再去使用这个地址是非常危险的。



## 函数指针

- 函数本身就是一段内存里面的代码，C 语言允许通过指针获取函数。

- 例: 变量`print_ptr`是一个函数指针，它指向函数`print()`的地址。函数`print()`的地址可以用`&print`获得。

  ```c
  void print(int a) {
    printf("%d\n", a);
  }
  
  void (*print_ptr)(int) = &print;
  
  // 函数指针定义时, 不仅要起名, 还要在后面的括号中标明传参类型
  ```

  - `(*print_ptr)`一定要写在圆括号里面，否则函数参数`(int)`的优先级高于`*`，整个式子就会变成`void* print_ptr(int)`。

  - 有了函数指针，通过它也可以调用函数。

    ```c
    (*print_ptr)(10);
    // 等同于
    print(10);
    ```



- C 语言还规定，函数名本身就是指向函数代码的指针，通过函数名就能获取函数地址。也就是说，`print`和`&print`是一回事。为了简洁易读，一般情况下，函数名前面都不加`*`和`&`。

  - 这种特性的一个应用是，如果一个函数的参数或返回值，也是一个函数，那么函数原型可以写成下面这样。

    - ```c
      int compute(int (*myfunc)(int), int, int);
      ```

    - 上面示例可以清晰地表明，函数`compute()`的第一个参数也是一个函数。



## 函数原型

- 函数必须先声明，后使用。由于程序总是先运行`main()`函数，导致所有其他函数都必须在`main()`函数之前声明。但是，`main()`是整个程序的入口，也是主要逻辑，放在最前面比较好。另一方面，对于函数较多的程序，保证每个函数的顺序正确，会变得很麻烦。

- C 语言提供的解决方法是，只要在程序开头处给出函数原型，函数就可以先使用、后声明。所谓函数原型，就是提前告诉编译器，每个函数的返回类型和参数类型。其他信息都不需要，也不用包括函数体，具体的函数实现可以后面再补上。

  - ```c
    int twice(int);
    
    int main(int num) {
      return twice(num);
    }
    
    int twice(int num) {
      return 2 * num;
    }
    ```

  - 函数原型包括参数名也可以，虽然这样对于编译器是多余的，但是阅读代码的时候，可能有助于理解函数的意图。

- 一般来说，每个源码文件的头部，都会给出当前脚本使用的所有函数的原型。





# struct

## 简介

- C 语言提供了`struct`关键字，允许自定义复合数据类型，将不同类型的值组合在一起。这样不仅为编程提供方便，也有利于增强代码的可读性。C 语言没有其他语言的对象（object）和类（class）的概念，struct 结构很大程度上提供了对象和类的功能。

- ```c
  // 定义
  struct fraction {
    int numerator;
    int denominator;
  };
  
  // "实例"化
  struct fraction f1;
  f1.numerator = 22;
  f1.denominator = 7;
  ```

  

- 除了逐一对属性赋值，也可以使用大括号，一次性对 struct 结构的所有属性赋值。
  - 大括号里面的值的顺序，必须与 struct 类型声明时属性的顺序一致。否则，必须为每个值指定属性名。
  - 若初始化的属性少于声明时的属性，这时剩下的那些属性都会初始化为`0`。
  - 声明变量以后，仍可以修改某个属性的值。

```c
struct car {
  char* name;
  float price;
  int speed;
};

struct car saturn = {"Saturn SL/2", 16000.99, 175};

struct car saturn = {.speed=172, .name="Saturn SL/2"};

saturn.speed = 168;
```





- struct 的数据类型声明语句与变量的声明语句，可以合并为一个语句。
  - 如果类型标识符只用在这一个地方，后面不再用到，这里可以将类型名省略 (匿名类型, 只有一个"实例")。
  - 与其他变量声明语句一样，可以在声明变量的同时，对变量赋值。

```c
struct b {
  char title[500];
  char author[100];
  float value;
} b1 = {"Harry Potter", "J. K. Rowling", 10.0},
  b2 = {"Cancer Ward", "Aleksandr Solzhenitsyn", 7.85};
```



- `typedef`命令可以为 struct 结构指定一个别名，这样使用起来更简洁。

  - 这样声明"实例"的时候就不需要struct关键字了!

  ```
  typedef struct cell_phone {
    int cell_no;
    float minutes_of_charge;
  } phone;
  
  phone p = {5551234, 5};
  ```

  

- 指针变量也可以指向`struct`结构。

  - 示例中，变量`b1`是一个指针，指向的数据是`struct book`类型的实例。

  ```c
  struct book {
    char title[500];
    char author[100];
    float value;
  }* b1;
  
  // 或者写成两个语句
  struct book {
    char title[500];
    char author[100];
    float value;
  };
  struct book* b1;
  ```

  



## struct 的复制

- struct 变量可以使用赋值运算符（`=`），复制给另一个变量，这时会生成一个全新的副本。系统会分配一块新的内存空间，大小与原来的变量相同，把每个属性都复制过去，即原样生成了一份数据。这一点跟数组的复制不一样，务必小心。

  ```c
  struct cat { char name[30]; short age; } a, b;
  
  strcpy(a.name, "Hula");
  a.age = 3;
  
  b = a;
  b.name[0] = 'M';
  
  printf("%s\n", a.name); // Hula
  printf("%s\n", b.name); // Mula
  // 变量b是变量a的副本，两个变量的值是各自独立的，修改掉b.name不影响a.name。
  // 当然, name如果是指针的话, 那还是会影响的
  ```

  

- 这种赋值要求两个变量是同一个类型，不同类型的 struct 变量无法互相赋值。
- C 语言没有提供比较两个自定义数据结构是否相等的方法，无法用比较运算符（比如`==`和`!=`）比较两个数据结构是否相等或不等。





## struct 指针

- 如果将 struct 变量传入函数，函数内部得到的是一个原始值的副本。

- 通常情况下，开发者希望传入函数的是同一份数据，函数内部修改数据以后，会反映在函数外部。而且，传入的是同一份数据，也有利于提高程序性能。这时就需要将 struct 变量的指针传入函数，通过指针来修改 struct 属性，就可以影响到函数外部。

  ```c
  #include <stdio.h>
  
  struct turtle {
    char* name;
    char* species;
    int age;
  };
  
  void happy(struct turtle* t) {
      // 函数内部必须使用(*t).age的写法，从指针拿到 struct 结构本身。
      // (*t).age不能写成*t.age，因为点运算符.的优先级高于*。
      // *t.age这种写法会将t.age看成一个指针，然后取它对应的值，会出现无法预料的结果。
      (*t).age = (*t).age + 1;
  }
  
  int main() {
    struct turtle myTurtle = {"MyTurtle", "sea turtle", 99};
    happy(&myTurtle);
    printf("Age is %i\n", myTurtle.age);  // 输出100
    return 0;
  }
  
  ```

  - `(*t).age`这样的写法很麻烦。C 语言就引入了一个新的箭头运算符（`->`），可以从 struct 指针上直接获取属性，大大增强了代码的可读性。

    ```c
    void happy(struct turtle* t) {
      t->age = t->age + 1;
    }
    ```

    

