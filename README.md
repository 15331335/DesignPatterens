# Design Patterens For Myself

设计模式是常见问题的有效解决方法。



## 在此之前

- 这是个人学习笔记，内容的伪代码均为 free style 以方便理解，并不遵循特定语法规则。
- 个人认为这些设计模式的优势通常与项目规模正相关，而内容中采用的例子可能过于简单容易出现「过度设计」的误解。
- 结构和内容参考了 [**design-patterns-for-humans**](https://github.com/kamranahmedse/design-patterns-for-humans) 和 [**菜鸟教程的设计模式**](http://www.runoob.com/design-pattern/design-pattern-tutorial.html) 等有用的链接。
- 6 + 7 + 10 = 23



## Creational

这类设计模式关注：

- 如何实例化
- 如何组织对象



### 简单工厂

首先来看一个接口和一个实现类：

```java
interface Person {
    func getName();
}

class Worker implements Person {
    var name;
    func __construct(n) { this->name = n; }
    func getName() { return this->name; }
}
```

下面是简单工厂类：

```java
class SimpleFactory {
    func employWorker(name) {
        return new Worker(name);
    }
}
```

显然简单的事情变复杂了，这种时候我们一般直接创建对象而不是通过工厂。而工厂模式的优势是在于**创建一个对象的时候涉及逻辑判断**的情况，如下：

```Java
interface Person { ... }

class Worker implements Person { ... }
class Manager implements Person { ... }

class Factory {
    func employ(type) {
        if (type) {
            return new Worker();
        } else {
            return new Manager();
        }
    }
}
```

进一步而言，工厂可以封装从特征推导类型的逻辑。这样的好处是在**经常创建不定类型的对象**时不用重复太多的代码。



### 工厂方法

同样地，先来看下面「老师」的接口和实现类：

```java
interface Teacher {
    func teach();
}
class ChineseTeacher implements Teacher {
    func teach() { /* Chinese */ }
}
class EnglishTeacher implements Teacher {
    func teach() { /* English */ }
}
```

接着是一个「课程」的抽象类及其继承：

```java
abstract class Course {
    // factory method
    abstract func getTeacher();
    
    func take() {
        this->getTeacher->teach();
    }
}

class ChineseCourse extends Course {
    func getTeacher { return new ChineseTeacher(); }
}
class EnglishCourse extends Course {
    func getTeacher { return new EnglishTeacher(); }
}
```

那么就可以开始「上课」了：

```java
var C = new ChineseCourse();
C->take()
```

从调用链来看，我们关注中间的工厂方法。对比工厂方法和简单工厂：前者所谓的工厂是类的一个方法，而后者工厂本身就是一个类；两者类似的情况，一个类的方法所依赖的子类在运行时动态地确定。



### 抽象工厂

简而言之，工厂们的工厂。下面是一个现实世界的例子：

- 门（接口）有两种（实现类）：木门和铁门。
- 修理工（接口）有对应的两种（实现类）：焊工和木匠。
- 那么门的工厂也有两种：一种是可以**制作**木门和提供木门**修理**服务的工厂，另一种是铁门的。

```java
interface DoorFactory {
    public func makeDoor() : Door;
    public func callRepairman() : Repairman;
}
```

从例子中我们可以推断这个模式的使用场景：多种对象有依赖关系。换句话说，这样的抽象工厂其实是在组装多个工厂的产品。



### 建造者

考虑这样一个对象的构造函数：多可选参数或多步骤。例如，一杯可定制的「咖啡」如下：

```java
class Coffee {
    func __construct(size, sugar = false);
}
```

对于这个类，我们将其多个构造参数转换成一个建造者：

```Java
class Coffee {
    var size, sugar = false;
    func __construct(builder) { // builder is a CoffeeBuilder
        this->size = builder->size;
        this->sugar = builder->sugar;
    }
}
```

抽象出构造者类的好处在于分离出构造参数的控制方法：

```Java
class CoffeeBuilder {
    var size, sugar = false, ice = false;
    
    func __construct(s) { this->size = s; }
    func addSugar() { this->sugar = true; }
    func addIce() { this->ice = true; }
    
    func build() {
        return new Coffee(this);
    }
}
```

这样我们就可以优雅地喝上一小杯咖啡了：

```java
var coffee = (new CoffeeBuilder(small))->addSugar()->build();
```

从整个流程来看，这样做可以避免构造函数的冗长，而区别于工厂的单步构造，建造者是针对多步的。



### 原型

可以这么说，把一个对象作为原型，其他的对象可以**根据原型来克隆**或构造。

```Java
var prototype = new Sheep(name);
prototype->getName();  // name
var cloned = prototype->clone();
cloned->setName('Dolly');
cloned->getName();  // 'Dolly'
```

直观感受就是对象的深拷贝。



### 单例

一句话，类的对象只有一个。

- 作为一种全局状态，好处是避免类的过度使用。
- 可能使得代码高耦合。

常见的做法是：

```java
final class Life {
    private:
    	var instance = new Life();  // Always static!
    	func __construct() { /* hide */ }
    	func __clone() { /* disable */ }
   	public:
    	func getInstance() {
            return this->instance
    	}
}
```

使用的时候：

```java
Life mine = Life.getInstance();  // rather than 'new Life()'
```



## Structural

这类设计模式关注：

- 对象的组成或实体的关联
- 组件化



### 适配器

假设在一个游戏中，「猎人」可以狩猎兔子和狗：

```java
interface Prey { func flee(); }

class Rabbit implements Prey {
    func flee() { /* run */ }
}
class Dog implements Prey {
    func flee() { /* run */ }
}

class Hunter {
    func hunt(prey) { prey->flee(); }
}
```

当游戏需要允许狩猎从未成为过猎物的老鹰时：

```java
class Hawk {
    func fly() { /* fly */ }
} 

// Adapter
class HawkAdapter implements Prey {
    var hawk;
    func __construct(hawk) { this->hawk = hawk; }
    func flee { this->hawk->fly(); }
}
```

从适配器的实现上不难发现，其内部维护了一个需要适配的新对象，并实现了原接口的方法，适配的方法就是指定原接口调用新对象的方法。



### 桥接

这个模式的思想在于**选择组成**而非继承：

```Java
interface painting {
    func __construct(style)
}
class Landscape implements painting {
    var style;
    func __construct(style) { this->style = style; }
}

interface Style { /* ... */ }
class Oil implements Style { /* ... */ }
class Wash implements Style { /* ... */ }
```

上面的例子很好理解，分离了抽象和现实；如果在抽象聚合关联关系时选择继承，这样不利于解耦。



### 组合

组合就是一个对象里面包含了许多同类的子对象，并为子对象进行统一的操作。

```java
interface Employee {
    func __construct(name, salary);
    func get*();  // getter
    func set*();  // setter
}

class Developer implements Employee { /* ... */ }
class Developer implements Employee { /* ... */ }

class Organization {
    var employees = [];
    func employ(name) {
        this->employees->push(new Employee(name));
    }
    func listEmployees() {  // in a uniform manner
        for (employee in this->employees) {
            employee->getName();
        }
    }
}
```

一般的使用场景特征：操作简单，并且自由增删



### 装饰器

装饰器的目的在于运行时包裹对象从而动态改变其行为。

```java
interface Book {
    func getDescription();
}

class OriginalBook implements Book {
    func getDescription() {
        return 'It is original.';
    }
}

// add-ons (decorators)
class ForeignBook implements Book {
    var book;
    func __construct(book) { this->book = book; }
    func getDescription() {
        return this->book->getDescription() + ' Foreign.';
    }
}
class IllustratedBook implements Book {
    var book;
    func __construct(book) { this->book = book; }
    func getDescription() {
        return this->book->getDescription() + ' Illustrated.';
    }
}
```

在使用时，可以不断地添加对于「书」的描述：

```Java
var book = new OriginalBook();
book = new ForeignBook(book);
book->getDescription();  // 'It is original. Foreign.'
book = new IllustratedBook(book);
book->getDescription();  // 'It is original. Foreign. Illustrated.'
```

不想增加很多子类而扩展类的时候可以使用装饰器。



### 外观

隐藏复杂系统，提供简单接口。

```java
class Computer {
    func stepA() { /* return this; */ }
    func stepB() { /* ... */ }
    func stepC() { /* ... */ }
}

class ComputerFacade {
    var computer;
    func __construct(computer) { this->computer = computer; }
    func turnOn() {
        this->computer->stepA();
        this->computer->stepB();
        this->computer->stepC();
        // this->computer->stepA()->stepB()-stepC();
    }
}
```

这是一个典型的例子，电脑的启动过程包含诸多系统步骤（如启动处理器、启动存储器等），而暴露给用户的只有一个操作，就是按下电源键。



### 享元

思想是通过**缓存**的方式减少存储或计算的开销。

```java
class Tool { /* type & ... */ }

// Anything that will be cached is flyweight.
class Toolbox {
    var tools = [];
    func getTool(type) {
        if (tools[type]->isEmpty()) {
            this->tools[type] = new Tool[type];
        }
        return this->tools[type];
    }
}

class Repairman {
    var toolbox;
    func __construct(toolbox) { this->toolbox = toolbox; }
    func check(type) {
        this->toolbox->getTool(type);
    }
    // repair
}
```

这个现实的例子是：修理工有一个工具箱，每当需要一种工具时先检查工具箱，如果没有则购买新的并放进去，这样以后可以反复使用不用每次修理都购买新的工具。



### 代理

典型解释是一个类代表另一个类的功能。

下面的例子描述了使用代理购买东西的情形，可以看到我们只需要付钱收货即可，前前后后的繁琐事情都在代理中被处理了。

```java
interface Abstract { func buy(something); }
class Real implements Abstract {
    fun buy(something) { /* pay */ }
}
class Prxoy implements Abstract {
    var real;
    func __construct(real) { this->real = real; }
    fun buy(something) {
        // sth. tedious before
        this->real->buy(something);
        // sth. tedious after
    }
}
```

重点在于：代理持有真实对象的引用。不同于类的简单组合，代理与真实对象都是同一个抽象接口的实现。



## Behavioral

这类设计模式关注：

- 对象间职责的分配
- 信息交流



### 责任链

通过创建一条对象链，使得请求沿着链找到合适的响应方法。

```Java
abstract class Account {
    var balance;
    var next;
    func setNext(account) { this->next = account; }
    func pay(amount) {
        if (this->balance >= amount) { /* pay it */ }
        else if (this->next) {
            this->next->pay(amount)
        } else { /* not afford */ }
    }
}

class PocketMoney extends Account {
    var balance;
    func __construct(money) { this->balance = money; }
}
class BankCard extends Account { /* ... */ }
```

当我们支付的时候，如果零钱不够就用银行卡：

```java
var pocketmoney = new PocketMoney(200);
var bankcard = new BankCard(500);

// a chain: $PocketMoney->$BankCard
pocketmoney->setNext(bankcard);

pocketmoney->pay(150);  // use PocketMoney
pocketmoney->pay(300);  // use BankCard
```

在这种模式中：

- 通常每个接收者都包含对另一个接收者的引用。
- 如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推。



### 命令

通过封装动作对 `client` 和 `receiver` 解耦：

```java
class Receiver {
    func add() { /* +1 */ }
    func subtract() { /* -1 */ }
}

interface Command {
    func execute();
    func undo();
    func redo();
}
class Addition implements Command {
    var receiver;
    func __construct(receiver) { this->receiver = receiver; }
    func execute() { this->receiver->add(); }
    func undo() { this->receiver->subtract(); }
    func redo() { this->execute(); }
}
class Subtraction implements Command { /* ... */ }
```

类似于遥控器的远程控制：

```Java
class Invoker {
    func submit(command) {
        command->execute();
    }
}
```

这样就可以通过构造命令并发送来控制接收端：

```java
var receiver = new Receiver();
var addition = new Addition(receiver)
var invoker = new Invoker()
invokere->submit(addition);
```

一个常见的使用场景就是上述存在 `undo` 操作的事务。



### 迭代器

一种访问对象元素的方式，关键在于无需知道底层的表示。

```Java
interface Iterator {
    func hasNext();
    func getNext();
}

interface Container {
   	func getIterator();
}
```

下面实现一个支持迭代访问的类：

```Java
class Repository implements Container {
    var elements = [ /* ... */ ];
    func getIterator() {
        return new RepoIterator();
    }
    
    private class RepoIterator implements Iterator {
        var index;
        func hasNext() {
            if (index < that->elements->len()) return true;
            else return false;
        }
        func getNext() {
            if (this->hasNext()) return that->elements[index++];
            else return NULL;
        }
    }
}
```

迭代器的实现有赖于具体的语法规则。



### 中介者

目的是降低类间交互造成的耦合。

典型的聊天室例子：

```java
// Mediator
class ChatRoom {
    func showMessage(user, message) {
        // show 'time, username & message'
    }
}

class User {
    var name, chatMediator;
    func __construct(name, chatMediator) {
        this->name = name;
        this->chatMediator = chatMediator;
    }
    func send(message) {
        this->chatMediator->showMessage(this, message);
    }
}
```

用户可以直接在聊天室中通信，注意到每个用户包含同样的中介者引用，这样网状的通信结构转换成了星型结构。



### 备忘录

在游戏中的存档功能：

```java
class Memento {
    var content;
    func __construct(content) { /* ...*/ }
    func getContent() { /* ... */ }
}

class Game {
    var temp, saved;  // Memento
    func load() { return this->saved->getContent(); }
    func save() { this->saved = new Memento(temp); }
}
```

合适的使用场景是：

- 需要保存/恢复数据的相关状态场景
- 提供一个可回滚的操作



### 观察者

这是对象之间的一种依赖：一个对象状态改变，告知它的观察者。

在实现上不难理解，只要对象持有观察者引用即可：

```java
class Obj {
    var state, observers = [];
    func notify() {
        for (observer in this->observers) {
            observer->update(this->state);
        }
    }
    func change(state) {
        this->state = state;
        this->notify();
    }
}
```

建立一套触发机制，注意避免循环依赖。



### 访问者

在访问者中动态地修改关于对象的执行逻辑，而不修改对象。

```Java
interface Visitee {
    func accept(operation);
}

interface Visitor {
    func visitButton(button);
    func visitSlider(slider);
    // overload: visit($element);
}
```

其中 `Visitor` 的访问方法可以根据语法规则使用重载。

```Java
class Button implements Visitee {
    func click() { /* ... */ }
    func accept(operation) {
        operation->visitButton(this);
    }
}
class Slider implements Visitee {
    func drag() { /* ... */ }
    func accept(operation) {
        operation->visitSlider(this);
    }
}

class UIOperation implements Visitor {
    func visitButton(button) { button->click(); }
    func visitSlider(slider) { slider->drag(); }
}
```

对于 UI 控件进行功能统一：

```Java
var operation = new UIOperation;
// ...
button->accept(operation);  // click button
slider->accept(operation);  // drag slider
```

上面是一个常见的使用场景，解决稳定的数据结构和易变的操作耦合问题。



### 策略

允许切换算法。

```java
interface Strategy {
    func sort(data);
}

class BubbleSort implements Strategy { /* ... */ }
class QuickSort implements Strategy { /* ... */ }
```

我们需要抽象上下文来封装策略引用：

```java
class Context {
    var strategy;
    func __construct(algorithm) {  // strategy is algorithm
        this->strategy = algorithm;
    }
    func sort(data) {
        return this->strategy->sort(data);
    }
}
```

这样在运行时根据情况生成包含策略算法的实例：

```java
var sorter = new Context(new BubbleSort());
// or 'var sorter = new Context(new QuickSort());'
```

可以避免因为使用条件语句所带来的复杂和难以维护。



### 状态

当状态改变时，改变对象的行为。

这个设计模式很好理解也非常常见，实现的方法是：

- 对于状态设计一个接口，并允许修改
- 封装状态在对象中维护

```java
interface State {
    func action();
}

class StartState implements State { /* ... */ }
class EndState implements State { /* ... */ }

class Obj {
    var state;
    func __construct(state) { this->state = state; }
    func setState(state) { this->state = state; }
    func doSomething() { this->state->action(); }
}
```

一句话，类的行为是基于其状态的。



### 模板方法

在抽象类实现通用的方法，而其他的步骤在子类实现。

下面是一个很好的自动化构建例子：

```java
abstract class Builder {
    // Template method
    func build() {
        this->test()->lint()->assemble()->deploy();
    }

    func test();
    func lint();
    func assemble();
    func deploy();
}
```

采用同样的构建流程，而前后端的构建细节不同：

```Java
class FrontEndBuilder extends Builder { /* implementations */ }
class BackEndBuilder extends Builder { /* implementations */ }

// usage
var backend = new BackEndBuilder();
backend->build();
```

可以看到：定义一个操作中的算法骨架，而将一些步骤延迟到子类中，使得子类不改变算法结构即可重定义该算法的某些特定步骤。





## 写在后面

- 受限于个人知识，内容难免差错，如果有问题请在 **issue** 中提出。
- 对于设计模式的使用域问题没有过多强调，希望只要代码优雅即可。
- **Star** ☕️
