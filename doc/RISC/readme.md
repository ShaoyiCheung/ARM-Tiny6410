ARM设计思想与高效C编程
====

###RISC设计思想

ARM内核采用**RISC体系结构**。**RISC是一种设计思想，其目标是设计出一套能在高时钟频率下单周期执行，简单而有效的指令集。**   
RISC的设计重点在于由硬件执行的指令的复杂度，这是因为软件比硬件容易提供更大的灵活性和更高的智能。因此，RISC设计对编译器有更高的要求；相反，传统的复杂指令集的计算机（CISC）则更侧重于硬件执行指令的功能性，使CISC变得更复杂。

RISC设计思想主要由下面4个设计准则来实现：   

* **指令集**     
	RISC处理器减少了指令种类，每条指令的长度都是固定的，允许流水线在当前指令译码阶段去取其下一条指令；而CISC处理器中，指令的长度通常不固定，执行也需要多个周期。	
* **流水线**    
	在理想情况下，流水线每周期前进一步，可获得最高的吞吐率；而CISC指令的执行需要调用微代码的一个微程序。
* **寄存器**    
	RISC处理器拥有更多的通用寄存器，每个寄存器都可存放数据或地址。寄存器可为所有的数据操作提供快速的局部存储访问；而CISC处理器都是用于特定目的的专用处理器。
* **load-store结构**    
	处理器只能处理寄存器中的数据。   
	独立的load和store指令用来完成数据在寄存器和外部存储器之间的传送。因为访问存储器很耗时，所以把存储器访问和数据处理分开。这样有一个好处，那就是可反复地使用保存在寄存器中的数据，而避免多次访问存储器。相反，在CISC结构中，处理器能够直接处理存储器中的数据。
	
	
###ARM设计思想

为降低功耗，ARM处理器已被特殊设计成较小的核，较高的代码密度。   
ARM内核不是一个纯粹的RISC体系结构，这是为了使它能够更好的适应其主要应用领域-嵌入式系统。   
在某种意义上，甚至可以认为ARM内核的成功，正是因为它没有在RISC概念上沉入太深。    
**现在系统的关键并不在于单纯的处理器速度，而在于有效的系统性能和功耗。**

* 低功耗
* 面向嵌入式系统的指令集   
* 一些特定指令的周期数可变   
* Thumb 16位指令集
* 条件执行
* 增强指令

###ARM下高效C编程

1. C数据类型的有效用法    
	
	对于存放在寄存器中的局部变量，除了8位或16位的算数模运算符外，**尽量不要使用char和short类型**。而要**使用有符号或者无符号的int类型（4字节长）**。    
	**除法运算时使用无符号数执行速度更快**。    
	 对于存放在主存储器中的数组和全局变量，在满足数据大小的前提下，应尽可能使用**小尺寸的数据类型，这样做可以节省存储器的存储空间**。ARMv4体系结构可以有效的装载和存储所有宽度的数据，并可以**使用递增数组的指针来有效的访问数组**。对于short类型数组，要避免使用数组基地址的偏移，因为LDRH指令不支持偏移寻址。    
	 由于隐式或者显式的数据类型转换通常会有额外的指令周期开销，所以**在表达式中应尽量避免使用数据类型转换**。load和store指令一般不会产生额外的转换开销，因为load和store指令是自动完成数据类型转换的。    
	**对于函数参数和返回值应尽量避免使用char和short类型**。即使参数范围比较小，也应该使用int类型，以防止编译器做不必要的类型转换。
	
2. 高效的编写循环体

	**使用减计数到零的循环结构**，这样编译器就不需要分配一个寄存器来保存循环中止值，而且与0比较的指令也可以省略。
		
		unsigned int num = 100;
		while(num--)
		{
			//...
		}
	**使用无符号的循环计数值，循环继续的条件为i!=0而不是i>0**，这样可以保证循环开销只有两条指令。    
	如果事先知道循环体至少会执行一次，那么使用do-while循环要比for循环好，这样可以使编译器省去检查循环计数值是否为0的步骤。    
	展开重要的循环体可降低循环开销，但不要过度展开，如果循坏的开销对整个程序来说占的比例很小，那么循环展开反而会增加代码量并降低cache性能。       
	尽量使数组的大小是4或8的倍数，这样就可以容易地以2，4，8次等多种选择展开循环，而不需要担心剩余数组元素的问题。

3. 高效的寄存器分配

	应该**尽量限制函数内部循环所用局部变量的数目，最多不超过12个**，这样，编译器就可以把这些变量都分配给ARM寄存器。   
	可以引导编译器，通过查看是否属于最内层循环变量来确定某个变量的重要性。
	
4. 高效的调用函数

	**尽量限制函数参数不要超过4个**，这样函数调用的效率会更高。也可以将几个相关的参数组织在一个结构体中，**用传递结构体指针来代替多个参数**。    
	把比较小的被调用函数和调用函数放在同一个原文件中，并且要先定义，后调用，编译器就可以优化函数调用或者内联较小的函数。
	**对性能影响较大的重要函数可使用关键字_inline进行内联。**
	
5. 避免指针别名

	**不要依赖编译器来消除包含存储访问的公共子表达式**，而应建立一个新的局部变量来保存这个表达式的值，这样可以保证只对这个表达式求职一次。
	避免使用局部变量的地址，否则对这个变量的访问效率会比较低。
	
6. 高效的结构体安排

	结构体元素要按照元素的大小来排列，以最小的元素放在开始，最大的元素安排在最后。    
	避免使用很大的结构体，可以使用层次化的小结构体来代替。   
	为了提高可移植性，人工对API的结构体添加填充位，这样，结构体的安排将不会依赖于编译器。    