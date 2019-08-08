# 误区

关于DI存在许多误解，因为首先完全熟悉可能很棘手。 在完全熟悉之前需要时间和经验。

如上例所示，DI可用于轻松交换给定接口的不同实现（在此示例中，这是ISomeService）。 但是，这只是DI提供的众多好处之一。

更重要的是，使用像Zenject这样的依赖注入框架可以让您更轻松地遵循“[单一责任原则](http://en.wikipedia.org/wiki/Single_responsibility_principle)”。 通过让Zenject连接类，类本身可以专注于履行其特定职责。

DI新手的另一个常见错误是它们从每个类中提取接口，并在任何地方使用这些接口而不是直接使用该类。 目标是使代码更松散地耦合，因此认为绑定到接口比绑定到具体类更好是合理的。 但是，在大多数情况下，应用程序的各种职责具有实现它们的单个特定类，因此在这些情况下使用接口只会增加不必要的维护开销。 此外，具体类已经具有由其公共成员定义的接口。 一个好的经验法则是仅在类具有多个实现时创建接口，或者在将来打算实现多个实现的情况下（顺便说一下，这已知为[重用抽象原则](http://codemanship.co.uk/parlezuml/blog/?postid=934)）

其他好处包括：
* 可重构性 - 当代码松散耦合时，正确使用DI的情况下，整个代码库对更改更具弹性。 您可以完全更改代码库的某些部分，而不会对其他部分造成严重破坏。

* 鼓励使用模块化代码 - 使用DI框架时，您自然会遵循更好的设计实践，因为它会强制您考虑类之间的接口。

* 可测试性 - 编写自动化单元测试或用户驱动测试变得非常容易，因为它只是编写一个不同的“组合根”，它以不同的方式连接依赖项。 想只测试一个子系统？ 只需创建一个新的组合根。 Zenject还支持避免组合根本身的代码重复（使用安装程序 - 如下所述）。

另请参阅[此处](https://github.com/modesttree/Zenject#isthisoverkill)和[此处](https://github.com/modesttree/Zenject#zenject-philophy)以进一步讨论和使用DI框架的理由。