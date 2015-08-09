title: Python的类机制
date: 2015-08-06 22:20:09
category: Python
tags:
---
## 初识Python类机制

首先参照图(pic_01):
![Alt text](/img/python/class_object_001.png "pic_01")

我们可知Python在类机制的实现中引入了两个很纠结的概念：PyTypeObject和基类，从而引出了Class与Instance以及基类和子类的概念。

原以为这两个概念并不存在太大的区别，可在逐步了解Python的对象机制的过程中，我发现两者相辅相成，相互交织却在功能划分上又泾渭分明。

对于一套完整的类实现机制，其流程为:

1. 创建Class对象（`PyTypeObject`）
2. 实例化对象来得到Instance（`PyObject`）
3. 使用Instance

其中，Class对象对应了上图（pic_01）中的中间部分，Instance对象对应最右边，而在最左边还有一个 `<type, 'type'>` 对象，该对象在创建Class对象的过程中起了至关重要的作用。所以图(pic_01)更多地向我们展示了类机制的`is-instance-of`关系，即instance是怎么一步步创建出来的。我觉得这很重要，它帮助我们理清了Python类机制的层次关系，解答了Python中的先有鸡还是先有蛋的问题。通过这样的叙述关系，我们知道了Instance对象是怎么来的，同样地，我们也弄清楚了负责创建Instance的Class对象又是怎么被创建。这对于我们了解Python的类机制怎么从无到有以及将Class与Instance的关系抽象出来并与其他语言的类概念联系起来，都非常有帮助。

但上面的描述不够具体，忽略了很多的实现细节。要想从根本上理解Python的对象机制，我们不仅要从设计上理解各个概念之间的关系，并且我们还需要从实现上窥视所谓的类和对象的本质。换种说法就是怎样用其他语言来实现Python的类机制。

## 类机制的实现

简单的说，Instance对象就是一堆数据以及围绕这些数据进行操作的一组方法的集合。在类机制里面，我们称数据之为成员变量，称围绕这些数据进行操作的方法为成员方法。


### C++中的实现

在C++中，Instance对象无非就是一段固定的内存和一堆指令码的集合。内存用于存放数据，指令码用于操作内存中的数据以及一些逻辑操作。

在C++的世界里，Class仅仅是Instance的模板，它仅仅定义了Instance应该是什么样子的，有哪些成员变量，有哪些成员方法。这样纯粹的说明是不需要使用内存的（为了简化问题，这里不考虑static成员变量，话说static存放的内存空间也跟成员变量不一样=.=!!），所以在C++中，Class仅仅是一堆指令码的集合（指令码会从硬盘读取到内存，但因为概念不一样，所以区别于一般的存放程序数据的内存）。而Instance就不一样了，在Class的new函数（也是一堆指令码>.<）的指挥下开始申请内存然后初始化成员变量，并且通过函数指针使用Class中定义的成员方法来操作内存中的成员变量。

总结一下就是，在C++的实现中：

1. Class用于描述Instance的结构和方法，其本质是一堆指令码，包括用于操作Instance的结构的new、destroy等方法，以及提供给Instance使用的方法。
2. Instance是一段固定的内存和一堆指令码的集合，是面向对象的设计思想中的核心。其内存的申请和分配均由Class的new函数来负责，而成员方法均由Class来定义。

> 脑洞大开一下，我再补充个观点，未得到验证所以不见得对，请大家绕道。

> 由于C++是对C的补充和扩展，所以它的实现也是依托于C。但是C本身是面向过程的语言，只有function和variable。套用我先前所提到的逻辑，*Instance是一段固定的内存和一堆指令码的集合*，慢着，怎么这么耳熟，这不就是variable（一段固定的内存）+ function（一堆指令码）的组合嘛！但是variable不能提前定义啊，否则它的内存地址就确定下来了，并且不同的Instance应该拥有属于自己的variable集合，这样的动态分配是Instance本身所不具备的功能，需要在更高层完成；同样地，我们不能重复定义function，所有的Instance应该share相同的function集合，再加上继承、共私有机制的引进，Instance本身无法实现这样的需求，同样需要在更高层完成。所以这时候我们需要Class来完成这些工作。Class通过new函数来为新创建的Instance申请空间，并在申请到的空间中申明variable；紧接着variable，Class会在之后的内存中存放Instance需要的函数所对应的函数指针。每个函数会根据所分配的首地址以及成员变量的地址偏移来得到variable并对其操作。大概这样就能实现了吧，么么哒~

### Python中的实现

至于基类的继承，更多地是属性的重用，所以图(pic_01)并未很好地展现继承的层次关系。

以上是我对`is-instance-of`以及`is-kind-of`的初步理解。接下来，我将通过对Class对象以及instance对象创建过程的剖析来探究PyTypeObject。不可避免地，我们会涉及到类的继承，因为它们本来就都是整个Class对象的一部分。

图(pic_01)很好地展示了Python的类机制的层次关系，即`metaclass`负责创建其他的class对象，而这些被创建的class对象负责创建instance对象，所以也就出现了两层`is-instance-of`关系。也正是因为存在这两层的
`is-instance-of`，class对象和instance对象才能从无到有。我在阅读《Python源码剖析》的过程中，不曾看到哪个PyTypeObject的ob_type所指向的非`metaclass`，这样的机制极大地简化了Python的类机制，易于理解也易于实现。

**为了更贴近Python的语言风格，在接下来的描述中，如果涉及到概念，我将延续先前使用的Class&Instance这样的术语，但在涉及到Python的具体实现中我将会**

1. 用`PyTypeObject`来表示Class对象
2. 用`PyObject实例`来表示Instance对象
3. 用`PyObject`来泛指Python的所有对象（everything in python is a object）
4. 用`PyBaseObject_Type`来表示所有Python最基本的Class对象，所有的`PyObject`均继承自它。原型为`<type, 'object'>`
 
为了更好地展示如何通过`PyTypeObject`来创建`PyObject实例`，我在这里展示另外一张图(pic_02):
![Alt text](/img/python/class_object_002.png "pic_02")

标上序号的虚线箭头代表了创建整数对象的函数调用流程，可以概括为：首先`PyTypeObject`(在这里即为`PyInt_type`)的tp_new会被调用，该函数可以视为new操作符（为instance对象申请内存空间以及初始化工作），若tp_new为NULL，`PyTypeObject`对象会去寻找基类(`PyBaseObject_Type`)的tp_new函数。因为所有的`PyObject`都继承自`PyBaseObject_Type`，而`PyBaseObject_Type`的tp_new的实现不为NULL，所以任何`PyObject`总能找到tp_new函数来完成用`PyObject实例`的创建。在具体的实现中，tp_new函数会访问`PyTypeObject`的tp_basicsize来确定要申请的空间大小；当空间申请结束之后，tp_new会调用tp_init函数来完成初始化工作。

Python的对象创建机制比较复杂，但工作原理却相当明朗，即Class对象调用自身的tp_new，赋以自身的一些必要属性作为参数，来为Instance对象分配内存和初始化工作。所以才有我先前的结论：Python是由Class对象（`PyTypeObject`）来负责对象的实例化，其中，我们看到某一Class对象的tp_new的具体实现可以不是由自身给出，而是顺藤摸瓜，通过一系列的函数指针的传递和函数调用来确定最终的函数实现。

在这里我们可以看到继承的运用。继承更多地