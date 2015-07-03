# IoC（Inversion of Control）

IoC 是一个框架使用的设计原则，用来让开发者来扩展框架或使用它创建应用程序。其基本思路是，框架是知道程序员的对象，并对它们进行调用。

这和开发者使用API时调用的API代码相反。因此，框架翻转控制：不再是开发者的代码负责，而是框架基于某些机制进行调用。

你可能已经在这个原则的指导下进行开发了，即使你没有意识到这一点。

## IoC容器

IoC容器使用上面所述的（简言之）原则管理类。包括，它们的创建，销毁，生命期，配置和依赖关系。这样类并不需要获取并配置它们所依赖的类。这在系统中极大地减少了耦合，并且简化了重用和可测试性。

那些认为“IoC”与“IoC容器”是一个同义词的人导致了一些混乱。如前所述，IoC是一个更广泛的原则。

有时候人们认为“注入”就是IoC的全部，并认为这是IoC容器的主要目的。事实上，“注射”是一个步骤，解耦的手段，而不是主要目的。

## 外部资源

* [Blog post by Stefano Mazzocchi (Jan 22, 2004)](http://www.betaversion.org/~stefano/linotype/news/38/)
* [bliki article by Martin Fowler, which totally misses the point](http://www.martinfowler.com/articles/injection.html)