# 升级到gradle4.4-all后，library中的jar包无法被主module引入的问题

**首先说明：**

api和implementation两种依赖的不同点在于：它们声明的依赖其他模块是否能使用。  
api：当其他模块依赖于此模块时，此模块使用api声明的依赖包是可以被其他模块使用  
implementation：当其他模块依赖此模块时，此模块使用implementation声明的依赖包只限于模块内部使用，不允许其他模块使用。

**所以**

在library的build文件中把引入jar包的引入语句中的compile或者implementation修改为api  
eg：api fileTree\(include: \['\*.jar'\], dir: 'libs'\)

**参考资料**

[Android Studio中Gradle依赖项配置变更](https://www.jianshu.com/p/d183c3b554e5)  
[Android Studio 3.0~3.x正式版填坑之路](https://www.jianshu.com/p/9b25087a5d7d)

