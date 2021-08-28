---
layout: post
title:  "Effective C++笔记"
date:   2021-08-28 14:26:00 +0800
categories: learning
---

## 条款01：视C++为一个语言联邦
* C: 区块（blocks）、语句（statements）、预处理器（preprocessor）、内置数据类型（built-in data types）、数组（arrays）、指针（pointers）等统统来自C
* OO: classes（包括构造函数和析构函数），封装（encapsulation）、继承（inheritance）、多态（polymorphism）、virtual函数（动态绑定）……等等
* Template
* STL: 对容器（containers）、迭代器（iterators）、算法（algorithms）以及函数对象（function objects）的规约有极佳的紧密配合与协调

## 条款02：尽量以const，enum，inline替换#define

## 条款03：尽可能使用const
* 令函数返回一个常量值，往往可以降低因客户错误而造成的意外，而又不至于放弃安全性和高效性

		class A {...};
		const A operator *(const A& l, const A& r);
		if (a * b = c) // 避免此类错误
* 两个成员函数如果只是常量性（constness）不同，可以被重载
* 移除 const 的那个动作只可以藉由const_cast完成，没有其他选择

## 条款04：确定对象被使用前已先被初始化
* 至于内置类型以外的任何其他东西，初始化责任落在构造函数（constructors）身上。规则很简单：确保每一个构造函数都将对象的每一个成员初始化。
* 如果成员变量是const或 references，它们就一定需要初值，不能被赋值
* C++规定，对象的成员变量的初始化动作发生在进入构造函数本体之前 => 初始化列表。总是使用成员初值列。这样做有时候绝对必要，且又往往比赋值更高效。
* 函数“内含static对象”的事实使它们在多线程系统中带有不确定性。再说一次，任何一种non-const static 对象，不论它是local或non-local，在多线程环境下“等待某事发生”都会有麻烦。处理这个麻烦的一种做法是：**在程序的单线程启动阶段**（single-threadedstartup portion）手工调用所有reference-returning函数，这可消除与初始化有关的“竞速形势（race conditions）”。

## 条款05：了解C++默默编写并调用哪些函数
* 惟有当这些函数(默认构造、拷贝构造、拷贝运算符、析构)被需要（被调用），它们才会被编译器创建出来。
* 编译器拒绝：如果打算在一个“内含 reference 成员”的class 内支持赋值操作（assignment），必须自己定义 copy assignment操作符。面对“内含const成员”（如本例之objectValue）的classes，编译器的反应也一样。

## 条款06：若不想使用编译器自动生成的函数，就该明确拒绝 Explicitly
* 关键是，所有编译器产出的函数都是public。为阻止这些函数被创建出来，你得自行声明它们，但这里并没有什么需求使你必须将它们声明为public。因此你可以将copy构造函数或copyassignment操作符声明为private并且没有定义。

		class Uncopyable {
		protected: 
		    Uncopyable() {}
		    ~Uncopyable() {}
		private:
		    Uncopyable(const Uncopyable&);
		    Uncopyable& operator=(const Uncopyable&);
		};
		class D: private Uncopyable

## 条款07：为多态基类声明virtual析构函数

## 条款08：别让异常逃离析构函数
* 如果程序遭遇一个“于析构期间发生的错误”后无法继续执行，“强迫结束程序”是个合理选项。
* 一般而言，将异常吞掉是个坏主意，因为它压制了“某些动作失败”的重要信息！
* 如果某个操作可能在失败时抛出异常，而又存在某种需要必须处理该异常，那么这个异常必须来自析构函数以外的某个函数。因为析构函数吐出异常就是危险，总会带来“过早结束程序”或“发生不明确行为”的风险。

## 条款09：绝不在构造和析构过程中调用virtual函数
* base class 构造期间 virtual 函数绝不会下降到derived classes阶层。取而代之的是，对象的作为就像隶属base类型一样。非正式的说法或许比较传神：在base class构造期间，virtual函数不是virtual函数。

## 条款10：令operator=返回一个 reference to*this

	A& operator=(const A& r) {
		if (this == &r) return *this;
		...
		return *this;
	}

## 条款11：在operator=中处理“自我赋值”

## 条款12：复制对象时勿忘其每一个成分
* 如果你为class添加一个成员变量，你必须同时修改copying函数。（你也需要修改class的所有构造函数（见条款4和条款45）
* 当你编写一个copying函数，请确保 （1）复制所有 local 成员变量，（2） 调用所有 base classes 内的适当的copying函数。

## 条款13：以对象管理资源
* 把资源放进对象内，我们便可倚赖 C++的“析构函数自动调用机制”确保资源被释放。
	* 获得资源后立刻放进管理对象（managing object）内。（Resource Acquisition Is Initialization；RAII） => 参考C++11 STL智能指针
	* 管理对象（managing object）运用析构函数确保资源被释放

## 条款14：在资源管理类中小心copying行为
* 复制 RAII 对象必须一并复制它所管理的资源，所以资源的 copying 行为决定RAII对象的copying行为。
* 普遍而常见的 RAII class copying 行为是：抑制 copying、施行引用计数法（referencecounting）。不过其他行为也都可能被实现。

## 条款15：在资源管理类中提供对原始资源的访问
* 对原始资源的访问可能经由显式转换(提供getter成员方法)或隐式转换(重载指针取值操作符 operator->和operator *)。一般而言显式转换比较安全，但隐式转换对客户比较方便。

## 条款16：成对使用new和delete时要采取相同形式
* 使用new时，有两件事发生。第一，内存被分配出来（通过名为operator new的函数，见条款49和条款51）。第二，针对此内存会有一个（或更多）构造函数被调用
* 使用 delete，也有两件事发生：针对此内存会有一个（或更多）析构函数被调用，然后内存才被释放（通过名为operator delete的函数，见条款51）

## 条款17：以独立语句将newed对象置入智能指针
* 以独立语句将 newed对象存储于（置入）智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄漏。 => 编译器优化导致不保证语句执行顺序

## 条款18：让接口容易被正确使用，不易被误用
* 理想上，如果客户企图使用某个接口而却没有获得他所预期的行为，这个代码不该通过编译；如果代码通过了编译，它的作为就该是客户所想要的。

## 条款19：设计class犹如设计type
* Class的设计就是type的设计。在定义一个新type之前，请确定：
	* 新type的对象应该如何被创建和销毁？
	* 对象的初始化和对象的赋值该有什么样的差别？
	* 新type的对象如果被passed by value（以值传递），意味着什么？
	* 什么是新type的“合法值”？
	* 你的新type需要配合某个继承图系（inheritance graph）吗？
	* 你的新type需要什么样的转换？
	* 什么样的操作符和函数对此新 type 而言是合理的？
	* 什么样的标准函数应该驳回？
	* 什么是新type的“未声明接口”（undeclared interface）？
	* 你的新type有多么一般化？
	* 你真的需要一个新type吗？
## 条款29：宁以pass-by-reference-to-const替换pass-by-value
* 高效：没有任何构造函数或析构函数被调用
* 以by reference方式传递参数也可以避免slicing（对象切割）问题

## 条款21：必须返回对象时，别妄想返回其reference
* 事情的真相是，任何函数如果返回一个reference指向某个local对象，都将一败涂地。

## 条款22：将成员变量声明为private
* 封装啦。如果你通过函数访问成员变量，日后可改以某个计算替换这个成员变量，而 class 客户一点也不会知道 class的内部实现已经起了变化。

## 条款23：宁以non-member、non-friend替换member函数
* 过多的member方法导致private的能力越大
* non-member、non-friend并不增加“能访问class内之private成分”的函数数量

## 条款24：若所有参数皆需类型转换，请为此采用non-member函数

## 条款25：考虑写出一个不抛异常的swap函数
> 这条感觉没有应用场景

## 额外信息
太快定义变量可能造成效率上的拖延；过度使用转型（casts）可能导致代码变慢又难维护，又招来微妙难解的错误；返回对象“内部数据之号码牌（handles）”可能会破坏封装并留给客户虚吊号码牌（dangling handles）；未考虑异常带来的冲击则可能导致资源泄漏和数据败坏；过度热心地inlining可能引起代码膨胀；过度耦合（coupling）则可能导致让人不满意的冗长建置时间（build times

## 条款26：尽可能延后变量定义式的出现时间 

## 条款27：尽量少做转型动作
* C++规则的设计目标之一是，保证“类型错误”绝不可能发生
* 新式转型较受欢迎。原因是：第一，它们很容易在代码中被辨识出来（不论是人工辨识或使用工具如grep），因而得以简化“找出类型系统在哪个地点被破坏”的过程。第二，各转型动作的目标愈窄化，编译器愈可能诊断出错误的运用。举个例子，如果你打算将常量性（constness）去掉，除非使用新式转型中的const_cast否则无法通过编译。

## 条款28：避免返回handles指向对象内部成分
* 两个教训：
	* 成员变量的封装性最多只等于“返回其reference”的函数的访问级别。
	* 如果const成员函数传出一个reference，后者所指数据与对象自身有关联，而它又被存储于对象之外，那么这个函数的调用者可以修改那笔数据

## 条款29：为“异常安全”而努力是值得的
* 带有异常安全性的函数会
	* 不泄漏任何资源
	* 不允许数据败坏
* 异常安全函数（Exception-safe functions）提供以下三个保证之一：
	* 基本承诺：数据正常，不保证逻辑
	* 强烈保证：如果抛出异常，程序状态不改变
	* 不抛掷保证：承诺绝不抛出异常

## 条款30：透彻了解inlining的里里外外
* inline造成的代码膨胀亦会导致额外的换页行为
* 大部分编译器拒绝将太过复杂（例如带有循环或递归）的函数 inlining，而所有对 virtual 函数的调用（除非是最平淡无奇的）也都会使inlining 落空
* 构造函数和析构函数往往是inlining 的糟糕候选人
* 大部分调试器面对 inline 函数都束手无策

## 条款31：将文件间的编译依存关系降至最低
* 支持“编译依存性最小化”的一般构想是：相依于声明式，不要相依于定义式。基于此构想的两个手段是Handle classes和Interface classes。

## 条款32：确定你的public继承塑模出is-a关系
* 以 C++进行面向对象编程，最重要的一个规则是：public inheritance（公开继承）意味 "is-a"（是一种）的关系。把这个规则牢牢地烙印在你的心中吧！

## 条款33：避免遮掩继承而来的名称
* derived classes内的名称会遮掩base classes内的名称。在public继承下从来没有人希望如此。
* 为了让被遮掩的名称再见天日，可使用 using 声明式或转交函数（forwarding functions）。

## 条款34：区分接口继承和实现继承
* 声明一个pure virtual函数的目的是为了让derived classes只继承函数接口
* 声明简朴的（非纯）impure virtual函数的目的，是让derived classes继承该函数的接口和缺省实现。
* 声明non-virtual函数的目的是为了令derived classes继承函数的接口及一份强制性实现。

## 条款35：考虑virtual函数以外的其他选择
* virtual函数的替代方案包括NVI手法及Strategy设计模式的多种形式。NVI手法自身是一个特殊形式的Template Method设计模式

## 条款36：绝不重新定义继承而来的non-virtual函数

## 条款37：绝不重新定义继承而来的缺省参数值

## 条款38：通过复合塑模出has-a或“根据某物实现出”
* 在应用域（application domain），复合意味 has-a （有一个）。在实现域（implementationdomain），复合意味is-implemented-in-terms-of（根据某物实现出）。

## 条款39：明智而审慎地使用private继承
* 尽可能使用复合，必要时才使用private继承
* Private继承意味is-implemented-in-terms of（根据某物实现出）。它通常比复合（composition）的级别低。但是当derived class需要访问protected base class的成员，或需要重新定义继承而来的virtual函数时，这么设计是合理的。
* 和复合（composition）不同，private继承可以造成empty base最优化。这对致力于“对象尺寸最小化”的程序库开发者而言，可能很重要。

## 条款40：明智而审慎地使用多重继承
* 使用 virtual 继承的那些 classes所产生的对象往往比使用non-virtual继承的兄弟们体积大，访问virtual base classes的成员变量时，也比访问non-virtual base classes的成员变量速度慢。种种细节因编译器不同而异，但基本重点很清楚：你得为virtual继承付出代价
* 关于virtual继承的忠告
	* 第一，非必要不使用virtualbases。平常请使用non-virtual继承。
	* 第二，如果你必须使用virtual base classes，尽可能避免在其中放置数据。这么一来你就不需担心这些classes身上的初始化（和赋值）所带来的诡异事情了。

## 额外
C++template机制自身是一部完整的图灵机（Turing-complete）：它可以被用来计算任何可计算的值。于是导出了模板元编程（template metaprogramming），创造出“在C++编译器内执行并于编译完成时停止执行”的程序


## 条款49：了解new-handler的行为
* 当 operator new抛出异常以反映一个未获满足的内存需求之前，它会先调用一个客户指定的错误处理函数，一个所谓的new-handler。
		
		namesapce std {
			typedef void (*new_handler)();
			new_handler set_new_handler(new_handler p) throw();
		}

* 一个设计良好的new-handler函数必须做以下事情
	*  让更多内存可被使用。
	*  安装另一个new-handler。
	*  卸除new-handler，也就是将null指针传给set_new_handler。
	*  抛出bad_alloc （或派生自bad_alloc）的异常。
	*  不返回，通常调用abort或exit。
* 参考：

		class NewHandlerHolder {
		public:
		    explicit NewHandlerHolder(std::new_handler nh): handler(nh) {}
		    ~NewHandlerHolder() {
		        std::set_new_handler(handler);
		    }
		private:
		    std::new_handler handler;
		    NewHandlerHolder(const NewHandlerHolder&);
		    NewHandlerHolder& operator=(const NewHandlerHolder&);
		}
		
		void* A::operator new(std::size_t size) throw(std::bad_alloc) {
		    NewHandlerHolder nh(std::set_new_handler(currentHandler));
		    return ::operator new(size);
		}

## 条款50：了解new和delete的合理替换时机
* 想要替换编译器提供的operator new或operator delete的理由
	* 用来检测运用上的错误
	* 强化效能：它们必须考虑破碎问题（fragmentation），这最终会导致程序无法满足大区块内存要求，即使彼时有总量足够但分散为许多小区块的自由内存。
	* 为了收集使用上的统计数据

## 条款51：编写new和delete时需固守常规
* perator new应该内含一个无穷循环，并在其中尝试分配内存，如果它无法满足内存需求，就该调用new-handler。它也应该有能力处理0 bytes申请。Class专属版本则还应该处理“比正确大小更大的（错误）申请”。
* 退出此循环的唯一办法是：内存被成功分配或new-handling函数做了一件描述于条款49的事情：让更多内存可用、安装另一个new-handler、卸除new-handler、抛出bad_alloc异常（或其派生物），或是承认失败而直接return。现在，对于new-handler为什么必须做出其中某些事你应该很清楚了。如果不那么做，operator new 内的while循环永远不会结束
