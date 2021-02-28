# 循环依赖是什么？

多个bean之间相互依赖，形成了一个闭环。比如A依赖于B、B依赖于C、C依赖于A

通常来说，如果问Spring容器内部如何解决循环依赖，一定是指默认的单例bean，属性相互引用的场景

![1614517753323](030-CircularDependencies/1614517753323.png)

![1614518396136](030-CircularDependencies/1614518396136.png)

**结论：我们AB循环依赖问题只要A的注入方式是setter且singleton, 就不会有循环依赖问题。**



