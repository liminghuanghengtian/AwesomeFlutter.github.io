# awesomeflutter

A new Flutter application.

## Getting Started

This project is a starting point for a Flutter application.

A few resources to get you started if this is your first Flutter project:

- [Lab: Write your first Flutter app](https://flutter.dev/docs/get-started/codelab)
- [Cookbook: Useful Flutter samples](https://flutter.dev/docs/cookbook)

For help getting started with Flutter, view our
[online documentation](https://flutter.dev/docs), which offers tutorials,
samples, guidance on mobile development, and a full API reference.

# Clean架构
## 1. 几个准则
1. 独立于框架-通过少量的代码形成clean架构的核心内容
2. 可测试性
3. 独立于UI-ui随意实现，只要基于给出的ui接口规范
4. 独立于数据库-数据库随意更换，业务规则不绑定具体的数据库实现，只依赖数据层的抽象
5. 独立于外部，不受外部影响

![AndroidClean](https://fernandocejas.com/assets/images/clean_architecture_clean_way_01.png)

## 2. todo-mvp-clean架构设计分析
### 2.1 UseCase（用例）抽象如何设计？
UseCase需要抽象如下三个部分：
- 输入，对应属性：`mRequestValues`（属性有相应的getter和setter）
- 输出，对应属性：`mUseCaseCallback`（输出的形式：回调的方式，且属性有相应的getter和setter）
- 处理，对应方法：`run()`（子类实现`executeUseCase()`方法）

![UseCase](https://note.youdao.com/yws/api/personal/file/WEB5f1b97e25e96f4402ab0187fa6a2a2c3?method=download&shareKey=8036b3d8e68f3bcaabaab0118f5d6874)
![Input&Output](https://note.youdao.com/yws/api/personal/file/WEB91fe6f5e075434cbd1568c9616009a8b?method=download&shareKey=60d7a5a7e882762a9333a0d668c4001c)

从UML用例图上可以看到UseCase是一个泛型类型，其类型参数Q的上界是`UseCase.RequestValues`，类型参数P的上界是`UseCase.ResponseValue`。另外可以注意到的一点是：`UseCase.UseCallback`也是一个泛型的设计，这里毋庸置疑，其类型参数显然将会是`UseCase.ResponseValue`的一个子类型，这一点从`setUseCaseCallback()`方法的参数列表可以看出。

#### 总结
UseCase的设计目标：UseCase的泛型设计组织起了一个可执行用例相关的输入、输出以及执行过程的模板。

### 2.2 UseCaseHandler（用例处理）设计
该处设计主要是封装对UseCase的处理过程。

![UseCaseHandler](https://note.youdao.com/yws/api/personal/file/WEB04bdd2bc31ff9fd38569d4480b75bd90?method=download&shareKey=2d0b32c16490f773ce83dc20bc4f31a9)

- UseCaseHandler是一个单例的实现，需要为全局所有的用例提供处理能力，每个用例都由其驱动处理。观察其参数列表，恰好组织了用例，输入，输出这三个部分。
- 通过`notifyResponse()`和`notifyError()`来反馈执行结果

### 2.3 UseCaseScheduler（用例调度器）的职责
- 调度用例的执行（对应其`execute()`方法）
- 通过`notifyResponse()`和`onError()`对外通知执行结果

### 2.4 UiCallbackWrapper职责

采用桥接模式，劫持UI层设置的用例执行结果监听，将用例执行结果通知的职责桥接给UseCaseHandler（用例处理器），将接口部分和实现部分分离，可以相对独立地加以改变。用例处理器又委托给调度器切到主线程，进行结果输出。


### 2.4 数据层如何抽象？
数据层的设计需要考虑如下目标：

1. 数据层的具体实现可以替换，要求设计一个数据层的抽象，不关心数据提供方的具体实现。此处应用依赖倒转原则，定义TaskDataSource接口，底层也只依赖TaskDataSource，数据层实现TaskDataSource通过ioc控制反转的思想注入实现到UseCase。
2. 移动端上数据主要来源于：本地数据库、网络中的远程数据、内存缓存三个部分，于是就拥有了三个数据提供方的实现TasksLocalDataSource、FakeTasksRemoteDataSource、LinkedHashMap以及聚合实现TasksRepository
3. 数据层是面向上层所有的交互操作设计的，保证一个最通用的策略
![DataSource](https://note.youdao.com/yws/api/personal/file/WEBdd08fe5b0f3e5e204eb8908f5a371b4a?method=download&shareKey=eab76470b188f1ab2237fee3edcaa20d)
![Repo](https://note.youdao.com/yws/api/personal/file/WEB84da9202fa4d72e2b33119c6885d7424?method=download&shareKey=ee5f85f34c4acf6535b37b53a9b7f199)

### 2.5 层次
表示层，领域层，模型层

### 2.6 代码结构
- 表示：任何的mvp
- 域：
- 数据：
- 
![CodeStruct](https://note.youdao.com/yws/api/personal/file/WEB3caba6c227a0752ed5568c1d8acd0e18?method=download&shareKey=a883691132ff0f5eb9a5958d096af861)

## 3. 依赖
源码依赖总是指向内层。越往内层移动抽象的等级越高。

# MVP架构
## 架构图
![arti](https://fernandocejas.com/assets/images/clean_architecture_clean_way_03.png)

# 共性
- 基于分层的思想设计，肉眼可见的优点是：投入最小的资源和成本来替换掉易变的部分（例如：数据层的实现）

# 组件粒度重构中的可行性&优缺点
## 可行性
可能存在的问题：
1. 组件间的通信如何解决？
通过观察者模式，在接口处定义注册监听和移除监听