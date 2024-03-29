## c语言记录

#### 0.语法

1. static——在使用 static 关键字修饰变量时，我们称此变量为静态变量。全局变量虽然属于静态存储方式，但并不是静态变量，它必须由 static 加以定义后才能成为静态全局变量。
   * 隐藏与隔离：全局变量的作用域是整个源程序，当一个源程序由多个源文件组成时，全局变量在各个源文件中都是有效的。通过在全局变量之前加上关键字 static 来实现，使全局变量被定义成为一个静态全局变量。这样就可以避免在其他源文件中引起的错误。也就起到了对其他源文件进行隐藏与隔离错误的作用，有利于模块化程序设计。
   * 保持变量持久性：当将局部变量声明为静态局部变量的时候，也就改变了局部变量的存储位置，即从原来的栈中存放改为静态存储区存放。这让它看起来很像全局变量，其实静态局部变量与全局变量的主要区别就在于可见性，静态局部变量只在其被声明的代码块中是可见的。
   * 默认初始化为0x00：在静态数据区，内存中所有的字节默认值都是 0x00。静态变量与全局变量也一样，它们都存储在静态数据区中，因此其变量的值默认也为 0。
   
2. const——常量一旦被创建后其值就不能再改变，所以常量必须在定义的同时赋值（初始化），后面的任何赋值行为都将引发错误。初始化常量可以使用任意形式的表达式。

   ```c
   #include <stdio.h>
   
   int getNum(){
       return 100;
   }
   
   int main(){
       int n = 90;
       const int MaxNum1 = getNum();  //运行时初始化
       const int MaxNum2 = n;  //运行时初始化
       const int MaxNum3 = 80;  //编译时初始化
       printf("%d, %d, %d\n", MaxNum1, MaxNum2, MaxNum3);
   
       return 0;
   }
   ```

   ```c
   const int *p1; //p1本身可以更改，但是指向的数据不能更改
   int const *p2; //p2本身可以更改，但是指向的数据不能更改
   int * const p3; //p3本身是只读的，指向的数据可以更改
   
   const int * const p4; //p4本身和所指向的数据都是只读的
   int const * const p5; //p5本省和所指向的数据都是只读的
   //const 离变量名近就是用来修饰指针变量的，离变量名远就是用来修饰指针指向的数据，如果近的和远的都有，那么就同时修饰指针变量以及它指向的数据。
   ```

   `const char *`和`char *`是不同的类型，不能将`const char *`类型的数据赋值给`char *`类型的变量。但反过来是可以的，编译器允许将`char *`类型的数据赋值给`const char *`类型的变量。

   这种限制很容易理解，`char *`指向的数据有读取和写入权限，而`const char *`指向的数据只有读取权限，降低数据的权限不会带来任何问题，但提升数据的权限就有可能发生危险。

#### 1.基本数据类型表示

1. 数据类型

   | 说明       | 字符型 | 短整型 | 整型 | 长整型 | 单精度浮点型 | 双精度浮点型 | 无类型 |
   | ---------- | ------ | ------ | ---- | ------ | ------------ | ------------ | ------ |
   | 数据类型   | char   | short  | int  | long   | float        | double       | void   |
   | 长度（32） | 1      | 2      | 4    | 4      | 4            | 8            | -      |

2. 整型长度：一种数据类型占用的字节数，称为该数据类型的长度。例如，short 占用 2 个字节的内存，那么它的长度就是 2,C语言并没有严格规定 short、int、long 的长度，只做了宽泛的限制：

   * short 至少占用 2 个字节
   * int 建议为一个机器字长。32 位环境下机器字长为 4 字节，64 位环境下机器字长为 8 字节
   * short 的长度不能大于 int，long 的长度不能小于 int
   * 总结起来，它们的长度（所占字节数）关系为：2 ≤ short ≤ int ≤ long

   > [大话C语言变量和数据类型](http://c.biancheng.net/cpp/html/2891.html)

   > [C语言中的整数（short,int,long）](http://c.biancheng.net/cpp/html/3092.html)
#### 2.union使用方法

1. union 维护足够的空间来置放多个数据成员中的“一种”，而不是为每一个数据成员配置空间，在union 中所有的数据成员共用一个空间，同一时间只能储存其中一个数据成员，所有的数据成员具有相同的起始地址.

2. 一个union 只配置一个足够大的空间以来容纳最大长度的数据成员.

   ```c
   union StateMachine
   {
      char character;
      int number;
      char *str;
      double exp;
   };
   ```

   以上例而言，最大长度是double 型态，所以StateMachine 的空间大小就是double 数据类型的大小。

3. 注意大小端模式对union类型数据的影响.

#### 3.标准头函数熟悉;
#### 4.布尔型判断;
#### 5.二叉树;
#### 6.软件复杂度;
#### 7.原码，反码，补码

6. 机器数 一个数在计算机中的二进制表示形式,  叫做这个数的机器数。机器数是带符号的，在计算机用一个数的最高位存放符号, 正数为0, 负数为1.

7. 真值 将带符号位的机器数对应的真正数值称为机器数的真值，例：0000 0001的真值 = +000 0001 = +1，1000 0001的真值 = –000 0001 = –1

8. 原码：就是符号位加上真值的绝对值, 即用第一位表示符号, 其余位表示值.第一位是符号位.八位二进制的取值范围：[1111 1111 , 0111 1111]即为[-127 , 127].  
    [+1]原 = 0000 0001  
    [-1]原 = 1000 0001

9. 反码：正数的反码是其本身,负数的反码是在其原码的基础上, 符号位不变，其余各个位取反.  
    [+1] = [00000001]原 = [00000001]反  
    [-1] = [10000001]原 = [11111110]反

10. 补码：正数的补码就是其本身负数的补码是在其原码的基础上, 符号位不变, 其余各位取反, 最后+1. (即在反码的基础上+1).
    [+1] = [00000001]原 = [00000001]反 = [00000001]补  
    [-1] = [10000001]原 = [11111110]反 = [11111111]补

11. why？
      [+1] = [00000001]原 = [00000001]反 = [00000001]补  
      [-1] = [10000001]原 = [11111110]反 = [11111111]补  
      1 - 1 = 1 + (-1) = [00000001]原 + [10000001]原 = [10000010]原 = -2  
      1 - 1 = 1 + (-1) = [0000 0001]原 + [1000 0001]原= [0000 0001]反 + [1111 1110]反 = [1111 1111]反 = [1000 0000]原 = -0  
      1 - 1 = 1 + (-1) = [0000 0001]原 + [1000 0001]原 = [0000 0001]补 + [1111 1111]补 = [0000 0000]补=[0000 0000]原 = 0  
      (-1) + (-127) = [1000 0001]原 + [1111 1111]原 = [1111 1111]补 + [1000 0001]补 = [1000 0000]补
>  [原码, 反码, 补码 详解](http://www.cnblogs.com/zhangziqiu/archive/2011/03/30/ComputerCode.html)

#### 8.位运算
1. 按位与运算通常用来对某些位清 0，或者保留某些位。例如要把 n 的高 16 位清 0 ，保留低 16 位，可以进行n & 0XFFFF运算（0XFFFF 在内存中的存储形式为 0000 0000 -- 0000 0000 -- 1111 1111 -- 1111 1111）
    1. &是根据内存中的二进制位进行运算的，而不是数据的二进制形式；其他位运算符也一样.  
    2. 以-9&5为例，-9 的在内存中的存储和 -9 的二进制形式截然不同：  
    1111 1111 -- 1111 1111 -- 1111 1111 -- 1111 0111  （-9 在内存中的存储）  
    -0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 1001  （-9 的二进制形式，前面多余的 0 可以抹掉）
    3. 9 & 5可以转换成如下的运算：  
    0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 1001  （9 在内存中的存储）  
    & 0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 0101  （5 在内存中的存储）  
    0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 0001  （1 在内存中的存储）
    4. -9 & 5可以转换成如下的运算：  
    1111 1111 -- 1111 1111 -- 1111 1111 -- 1111 0111  （-9 在内存中的存储）  
    & 0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 0101  （5 在内存中的存储）  
    0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 0101  （5 在内存中的存储）
2. 按位或运算可以用来将某些位置 1，或者保留某些位.。例如要把 n 的高 16 位置 1，保留低 16 位，可以进行n | 0XFFFF0000运算（0XFFFF0000 在内存中的存储形式为 1111 1111 -- 1111 1111 -- 0000 0000 -- 0000 0000）.
    1. 9 | 5可以转换成如下的运算：  
    0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 1001  （9 在内存中的存储）  
    | 0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 0101  （5 在内存中的存储）  
    0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 1101  （13 在内存中的存储）
    2. -9 | 5可以转换成如下的运算：  
    1111 1111 -- 1111 1111 -- 1111 1111 -- 1111 0111  （-9 在内存中的存储）  
    | 0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 0101  （5 在内存中的存储）  
    1111 1111 -- 1111 1111 -- 1111 1111 -- 1111 0111  （-9 在内存中的存储）
3. 按位异或运算可以用来将某些二进制位反转。例如要把 n 的高 16 位反转，保留低 16 位，可以进行n ^ 0XFFFF0000运算（0XFFFF0000 在内存中的存储形式为 1111 1111 -- 1111 1111 -- 0000 0000 -- 0000 0000）.
    1. 9 ^ 5可以转换成如下的运算：  
    0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 1001  （9 在内存中的存储）  
    ^ 0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 0101  （5 在内存中的存储）  
    0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 1100  （12 在内存中的存储）
    2. -9 ^ 5可以转换成如下的运算：  
    1111 1111 -- 1111 1111 -- 1111 1111 -- 1111 0111  （-9 在内存中的存储）  
    ^ 0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 0101  （5 在内存中的存储）  
    1111 1111 -- 1111 1111 -- 1111 1111 -- 1111 0010  （-14 在内存中的存储）
4. 取反运算符~为单目运算符，右结合性，作用是对参与运算的二进制位取反.
    1. ~9可以转换为如下的运算：  
    ~ 0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 1001  （9 在内存中的存储）  
    1111 1111 -- 1111 1111 -- 1111 1111 -- 1111 0110  （-10 在内存中的存储）
    2. ~-9可以转换为如下的运算：  
    ~ 1111 1111 -- 1111 1111 -- 1111 1111 -- 1111 0111  （-9 在内存中的存储）  
    0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 1000  （9 在内存中的存储）
5. 左移运算符<<用来把操作数的各个二进制位全部左移若干位，高位丢弃，低位补0.如果数据较小，被丢弃的高位不包含 1，那么左移 n 位相当于乘以 2 的 n 次方.
    1. 9<<3可以转换为如下的运算：  
    << 0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 1001  （9 在内存中的存储）  
    0000 0000 -- 0000 0000 -- 0000 0000 -- 0100 1000  （72 在内存中的存储）
    2. (-9)<<3可以转换为如下的运算：  
    << 1111 1111 -- 1111 1111 -- 1111 1111 -- 1111 0111  （-9 在内存中的存储）  
    1111 1111 -- 1111 1111 -- 1111 1111 -- 1011 1000  （-72 在内存中的存储）
6. 右移运算符>>用来把操作数的各个二进制位全部右移若干位，低位丢弃，高位补 0 或 1。如果数据的最高位是 0，那么就补 0；如果最高位是 1，那么就补 1.如果被丢弃的低位不包含 1，那么右移 n 位相当于除以 2 的 n 次方（但被移除的位中经常会包含 1）.
    1. 9>>3可以转换为如下的运算：  
    &gt;&gt; 0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 1001  （9 在内存中的存储）  
    0000 0000 -- 0000 0000 -- 0000 0000 -- 0000 0001  （1 在内存中的存储）
    2. (-9)>>3可以转换为如下的运算：
    &gt;&gt; 1111 1111 -- 1111 1111 -- 1111 1111 -- 1111 0111  （-9 在内存中的存储）  
    1111 1111 -- 1111 1111 -- 1111 1111 -- 1111 1110  （-2 在内存中的存储）
    >  [C语言位运算](http://c.biancheng.net/cpp/html/101.html)

#### 9.大端模式和小端模式
1. 大端模式（little-endian）：字数据的高字节存储在低地址中，而字数据的低字节则存放在高地址中.

2. 小端模式（big-endian）：低地址中存放的是字数据的低字节，高地址存放的是字数据的高字节.

3. 地址向下生长，下端为高地址，上端为低地址.

4. 0x1234中，12为高字节，34为低字节.

5. 16bit宽的数0x1234在CPU内存中的存放方式（假设从地址0x4000开始存放）为：

1. Little-endian模式

    | 内存地址 | 0x4000 | 0x4001 |
    | -------- | ------ | ------ |
    | 存放内容 | 0x34   | 0x12   |
    
    2. Big-endian模式

    | 内存地址 | 0x4000 | 0x4001 |
    | -------- | ------ | ------ |
    | 存放内容 | 0x12   | 0x34   |

6. 32bit宽的数0x12345678在CPU内存中的存放方式（假设从地址0x4000开始存放）为：
    1. Little-endian模式  

    | 内存地址 | 0x4000 | 0x4001 | 0x4002 | 0x4003 |
    | -------- | ------ | ------ | ------ | ------ |
    | 存放内容 | 0x78   | 0x56   | 0x34   | 0x12   |
    2. Big-endian模式  

    | 内存地址 | 0x4000 | 0x4001 | 0x4002 | 0x4003 |
    | -------- | ------ | ------ | ------ | ------ |
    | 存放内容 | 0x12   | 0x34   | 0x56   | 0x78   |

    
    
8. 联合体union的存放顺序是所有成员都从低地址开始存放.

9. 判断方法

10. 第一种方法

     ```c
     /* 
      * 1: little-endian 
      * 0: big-endian 
      */  
     int check_endian()  
     {  
         int a = 1;  
         char *p = (char *)&a;  
       
         return (*p == 1);  
     }  
     
     //如果是大端，*p的结果是0
     ```

     

11. 第二种方法

       ```c
        /* 
         * 1: little-endian 
         * 0: big-endian 
         */  
        int check_endian()  
        {  
            union w  
            {  
                int a;  
                char b;  
            } c;  
            c.a = 1;  
            return (c.b == 1);  
        }  
        
        printf("%s\n", checkEndian() ? "little-endian" : "big-endian"); 
       ```

       

12. 大端和小端的转换

     ```c
      /*
       * 大小端转换
      */
      int big_litle_endian(int x)
      {
          int tmp;
          tmp = (((x)&0xff) << 24) + (((x >> 8) & 0xff) << 16) + (((x >> 16) & 0xff) << 8) + (((x >> 24) & 0xff));
          return tmp;
      }
     ```

     

#### 10.十六进制

**十六进制**（简写为*hex*或下标16）在[数学](https://zh.wikipedia.org/wiki/%E6%95%B0%E5%AD%A6)中是一种逢16进1的[进位制](https://zh.wikipedia.org/wiki/%E8%BF%9B%E4%BD%8D%E5%88%B6)。一般用数字0到9和字母A到F（或a~f）表示，其中:A~F表示10~15，这些称作**十六进制数字**。

现在的16进制则普遍应用在[计算机](https://zh.wikipedia.org/wiki/%E8%AE%A1%E7%AE%97%E6%9C%BA)领域，这是因为将4个位元（Bit）化成单独的16进制数字不太困难。1字节可以表示成2个连续的16进制数字。可是，这种混合表示法容易令人混淆，因此需要一些字首、字尾或下标来显示。

#### 11.ASCII

> [ASCII](https://zh.wikipedia.org/wiki/ASCII)

#### 12.字符串初始化

1. char str[10]="";
2. char str[10]={'\0'};
3. char str[10]; str[0]='\0';

如果花括号中提供的字符个数大于数组长度，则按语法错误处理；若小于数组 长度，则只将这些字符数组中前面那些元素，其余的元素自动定为空字符（即 '\0' )。

#### 13.交叉编译

1. arm-2008q3-linux

   ```sh
   export SDKTARGETSYSROOT=/opt/arm-2008q3-linux/arm-none-linux-gnueabi/libc
   export PATH=/opt/arm-2008q3-linux/bin:$PATH
   export CC="arm-none-linux-gnueabi-gcc --sysroot=$SDKTARGETSYSROOT"
   export CXX="arm-none-linux-gnueabi-g++ --sysroot=$SDKTARGETSYSROOT"
   export CPP="arm-none-linux-gnueabi-gcc -E --sysroot=$SDKTARGETSYSROOT"
   export AS="arm-none-linux-gnueabi-as "
   export LD="arm-none-linux-gnueabi-ld  --sysroot=$SDKTARGETSYSROOT"
   export GDB=arm-none-linux-gnueabi-gdb
   export STRIP=arm-none-linux-gnueabi-strip
   export RANLIB=arm-none-linux-gnueabi-ranlib
   export OBJCOPY=arm-none-linux-gnueabi-objcopy
   export OBJDUMP=arm-none-linux-gnueabi-objdump
   export AR=arm-none-linux-gnueabi-ar
   export NM=arm-none-linux-gnueabi-nm
   export M4=m4
   export CFLAGS=" -O2 -pipe -g -feliminate-unused-debug-types"
   export CXXFLAGS=" -O2 -pipe -g -feliminate-unused-debug-types"
   export LDFLAGS="-Wl,-O1 -Wl,--hash-style=gnu -Wl,--as-needed"
   export CPPFLAGS=""
   export ARCH=arm
   
   # export CROSS_COMPILE=arm-none-linux-gnueabi-
   ```

2. sysroot一般含有etc，lib，usr，sbin目录，usr目录下有include，lib，man。

3. cmake .. -DCMAKE_INSTALL_PREFIX:PATH=/opt/arm-2008q3-linux/arm-none-linux-gnueabi/libc/usr

   sh configure --prefix=/opt/arm-2008q3-linux/arm-none-linux-gnueabi/libc/usr --host=arm-linux

   make CC=""

   sudo -i

   source......

#### 14.CMake

1. 调试编译 cmake  -DCMAKE_BUILD_TYPE=Debug ..

#### 15.Debug

1. ulimit -c unlimited 
2. gcc core_demo.c -o core_demo -g

#### 16.内存管理总结

进程的地址空间是独立的，他们之间互不影响，好处：a.每一个进程的地址空间变大了，编写程序更容易；b.一个进程崩溃了，不会影响其他进程，提高了系统整体的稳定性。

由MMU（内存管理单元，物理器件）根据页表把虚拟内存地址转换成对应的物理内存地址。系统一般会创建一个专门的分区存放换出的内存数据，这个分区称为“交换分区”。

虚拟内存到物理内存的映射以页为最小单位映射，页通常为4KB大小，当应用程序访问的虚拟内存页不在物理内存里时，MMU产生一个缺页中断，挂起进程，缺页中断处理函数负责把相应数据从磁盘读入内存，然后唤醒挂起的进程。

内存分配方式：

1. 从静态存储区域分配，内存再编译的时候已经分配好了，这块内存再程序的整个运行期间都存在；
2. 从栈上创建，再执行函数时，函数内局部的存储单元都可以再栈上创建，函数执行结束时这些存储单元自动被释放，栈内存分配运算内置于处理器指令集中，效率很高，但是分配的内存容量有限；
3. 从堆上分配，亦称为动态内存分配，程序再运行的时候用malloc或new申请任意多内存，程序员自己负责在任何时候用free或delete释放内存。

常见内存错误：

1. 内存分配未成功，却使用了它。一般在使用内存前检查其指针是否为NULL，如果指针是函数的参数可以使用assert(p != NULL)尽心测试；如果使用malloc或new来申请内存，应该用if(p == NULL)来进行防错处理；
2. 内存分配虽然成功，但是尚未初始化就应用它。内存的缺省初始值是什么并无统一标准，尽管有些时候为0值，我们宁可信其有，所以无论何种方式创建数组，别忘了赋初始值，即使赋0值也不可省略，不要嫌麻烦。
3. 内存分配成功并且初始化，但是操作越界。
4. 忘记释放内存，造成内存泄漏，动态呢村的申请与释放必须配对，程序中malloc与free的使用次数一定要相同，否则肯定有错误。
5. 释放了内存却继续使用它。有三种情况：
   1. 程序中的对象调用关系过于复杂，实在难以搞清楚某个对象究竟是否已经释放了内存，此时应重新设计数据结构，从根本上解决对象管理的混乱局面。
   2. 函数的return语句写错了，注意不要放回指向“栈内存”的指针，因为该内存在函数体结束时，被自动销毁了；
   3. 使用free释放内存后，没有将指针设置为NULL，导致产生“野指针”。

内存管理规则：

1. 用malloc申请内存之后，应该立即检查指针时候为NULL。防止使用指值为NULL的内存；
2. 不要忘记数组和动态内存赋初值。防止将违背初始化的内存作为右值使用；
3. 避免数组或指针下标越界，特别要担心发生“多1”或者“少1”操作；
4. 动态内存的申请与释放必须配对，防止内存泄漏；
5. 用free释放了内存之后，立即将野指针设置为NULL，防止产生“野指针”。

指针与数组：

1. 数组要么在静态存储去被创建（如全局数组），要么在栈上被创建。数组名对应着（而不是指向了）一块内存，其地址与容量在生命周期内保持不变，只有数组的内容可以改变。
2. 指针可以随时指向任意类型的内存块，它的特征是“可变”，所以我们常用指针来操作动态内存，指针远不数组灵活，但更危险；
3. 不能对数组名进行直接复制与比较，复制使用memcpy，比较使用memcmp；
4. p=a不能把a的内容复制给指针p；而是把a的地址赋给p。

计算内存容量：

```c
char a[] = "hello world!";
char *p = a;
sizeof(a); //-> 13
sizeof(p); //-> 4

void func(char a[13]){
    sizeof(a); //-> 4 当数组作为函数的参数进行传递时，该数组自动退化为同类型的指针
}
```

指针参数：

指针参数也是值，c语言没有严格意义上的引用传递，只有值传递。编译器总是会为函数的每个参数制作临时副本，在函数内部修改指针参数的值，没有用，因为这只是一个副本，但是可以改变其指向内存块的值，所以可以用于输出参数。

特别注意在函数体内分配内存再把内存地址赋给指针参数，不但值为NULL，而且还会造成内存泄漏，因为没有对应的free（不可能有）。这种情况下，可以使用“指向指针”的指针，用指针的参数去申请内存。

```c
void get_memory(char **p, int num){
    *p = (char *)malloc(sizeof(char) * num);
}

main(void){
    char *str = NULL;
    get_memory(&str, 10);
}
```

野指针：

free只是把指针所指向的内存给释放掉，但是并没有把指针本身干掉，指针p被free以后地址仍然不变（非NULL），只是该地址对应的内存是垃圾，p成了野指针，通常会用if(p!=NULL)进行防错处理。但遗憾的是起不了作用，因为即便p不是NULL指针，它也不指向合法的内存块。

野指针不是NULL指针，成因1：指针变量没有被初始化，创建时不会自动生成NULL指针，而是会乱指，所以指针变量在创建时应该被初始化，要么设置为NULL，要么指向合法内存；成因2：指针被free后，没有设置为NULL，让人误以为是合法指针；成因3：指针操作超越了变量的作用范围（通常在c++中）。

指针消失了，并不表示它指向的内存会被自动释放，内存被释放了，并不表示指针会消亡或者称为NULL指针。

分配内存：

32位系统内存几乎不大可能耗尽，但是不加错误处理会导致程序质量很差。

malloc应当注意两个要素：类型转换和sizeof，malloc返回void*，所以需要显式类型转换；malloc中使用sizeof是良好的风格。

free因为事先知道指针的类型和容量，所以能正确的释放内存；如果p是NULL指针，无论free多少次都不会出问题，如果p不是NULL指针，那么free对p连续操作两次就会导致运行错误。

常见内存错误：1.内存泄漏；2.内存越界访问——除非你确定输入数据式在你控制之内的，否则不要使用strcpy、strcat和sprintf之类的函数；3.野指针；4.访问空指针；5.引用未初始化的变量；6.不清楚指针运算；7.结构的成员顺序变化引发的错误；8.结构的大小变化引发的错误；9.分配/释放不配对；10.放回指向临时变量的指针；11.试图修改常量；12.误解传值与传引用。13.符号重名；14.栈溢出；15.误用sizeof；16.字节对齐——程序内部用sizeof取得结构体大小就足够了，若数据在不同机器间传递就需要规定对其方式；17.字节顺序；18.多线程共享变量没有用valotile修饰——告诉编译器不要把变量优化到寄存器里；19.忘记函数的返回值。

#### 17.栈Stack

调用c语言函数时，参数按值传递，并从最后一个参数开始压栈。先压入最后一个参数，再压入倒数第二个参数，最后压入第一个参数，栈是向下增长的，先入栈的参数放在高地址，后入栈的参数放在低地址。

PC上普通线程的栈空间也有十多MB，一些嵌入式系统中，线程栈空间可能只有5KB，甚至小到256字节。

#### 18.线程

同一个进程中的多个线程，他们的内存空间是共享的（栈除外），一个线程对内存的修改，对所有线程都有效。

#### 19.程序数据段

未初始化的全局变量（.bss段）：bss段用来存放那些没有初始化或初始化位0的全局变量。bss类型的全局变量只占运行时内存空间，不占文件空间，运行周期内一直存在。

```bash
objdump -h bss.exe |grep bss
```

初始化过的全局变量（.data段）：data段用于存放那些初始化为非0值得全局变量。data类型的全局变量既占用文件空间，又占用内存空间，运行周期内一直存在。

```bash
objdump -h data.exe |grep \\.data
```

常量数据段（.rodata段）：ro代表只读；1.常量不一定在rodata里，又的立即数直接和指令编码在一起，放在代码段.text段中。2.对于字符串常量，编译器会自动去掉重复的字符串，保证一个字符串在一个可执行文件中只有一个副本。3.rodata是在多个进程间共享的，这样可以提高运行空间利用率。4.在有的嵌入式系统中，rodata放在ROM或者NOR闪存芯片里，运行时直接读取，无需加载到RAM。 5.在嵌入式Linux系统中，也可以通过一种叫做XIP（就地执行）得技术直接读取常量数据，而无需加载到RAM中。6.常量是不能修改的，修改常量在Linux下会出现段错误。

代码（.text段）：text段存放代码和部分整数常量，它与rodata段很相似，主要区别是text段时可执行的。

栈（stack）：栈是用来存放临时变量和函数参数的。通常情况下，栈是向下（低地址）增长的，每向栈中PUSH一个元素，栈顶就向低地址扩展，每从栈中POP一个元素，栈顶就向高地址回退。

堆（heap）：堆是最灵活的一种内存，它的生命周期完全由使用者控制。
