---
layout: post
title: "AutoFac使用方法总结:Part I"
date: 2012-04-06 23:07
comments: true
categories: 
---

AutoFac是.net平台下的IOC容器产品，它可以管理类之间的复杂的依赖关系。在使用方面主要是register和resolve两类操作。
这篇文章用单元测试的形式列举了AutoFac的常用使用方法：
<!-- more -->
<h3>注册部分</h3>
使用RegisterType进行注册
``` c#
		[Fact]
        public void can_resolve_myclass()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<MyClass>();

            IContainer container = builder.Build();
            var myClass = container.Resolve<MyClass>();
            Assert.NotNull(myClass);
        }
```

注册为接口
``` c#
		[Fact]
        public void register_as_interface()
        {
            var builder = new ContainerBuilder();
            builder.Register(c => new MyClass()).As<MyInterface>();

            IContainer container = builder.Build();
            Assert.NotNull(container.Resolve<MyInterface>());
            Assert.Throws(typeof (ComponentNotRegisteredException), () => container.Resolve<MyClass>());
        }
``` 

使用lambda表达式进行注册
``` c#
		[Fact]
        public void can_register_with_lambda()
        {
            var builder = new ContainerBuilder();
            builder.Register(c => new MyClass());

            IContainer container = builder.Build();
            var myClass = container.Resolve<MyClass>();
            Assert.NotNull(myClass);
        }
```
带构造参数的注册		

``` c#
		[Fact]
        public void register_with_parameter()
        {
            var builder = new ContainerBuilder();
            builder.Register(c => new MyParameter());
            builder.Register(c => new MyClass(c.Resolve<MyParameter>()));
            IContainer container = builder.Build();
            Assert.NotNull(container.Resolve<MyClass>());
        }	
```

带属性赋值的注册
``` c#
		[Fact]
        public void register_with_property()
        {
            var builder = new ContainerBuilder();
            builder.Register(c => new MyProperty());
            builder.Register(
                c => new MyClass()
                         {
                             Property = c.Resolve<MyProperty>()
                         });
            IContainer container = builder.Build();
            var myClass = container.Resolve<MyClass>();
            Assert.NotNull(myClass);
            Assert.NotNull(myClass.Property);
        }		
```
Autofac分离了类的创建和使用，这样可以根据输入参数（NamedParameter）动态的选择实现类。
``` c#
		[Fact]
        public void select_an_implementer_based_on_parameter_value()
        {
            var builder = new ContainerBuilder();
            builder.Register<IRepository>((c, p) =>
                                 {
                                     var type = p.Named<string>("type");
                                     if (type == "test")
                                     {
                                         return new TestRepository(); 
                                     }
                                     else
                                     {
                                         return new DbRepository();
                                     }
                                 }).As<IRepository>();

            IContainer container = builder.Build();
            var repository = container.Resolve<IRepository>(new NamedParameter("type", "test"));
            Assert.Equal(typeof(TestRepository),repository.GetType());
        }
```
AufoFac也可以用一个实例来注册，比如用在单例模式情况下：
``` c#
		[Fact]
        public void register_with_instance()
        {
            var builder = new ContainerBuilder();
            builder.RegisterInstance(MyInstance.Instance).ExternallyOwned();
            IContainer container = builder.Build();
            var myInstance1 = container.Resolve<MyInstance>();
            var myInstance2 = container.Resolve<MyInstance>();
            Assert.Equal(myInstance1,myInstance2);
        }
```
注册open generic类型
``` c#
		[Fact]
        public void register_open_generic()
        {
            var builder = new ContainerBuilder();
            builder.RegisterGeneric(typeof (MyList<>));
            IContainer container = builder.Build();
            var myIntList = container.Resolve<MyList<int>>();
            Assert.NotNull(myIntList);
            var myStringList = container.Resolve<MyList<string>>();
            Assert.NotNull(myStringList);
        }
```
对于同一个接口，后面注册的实现会覆盖之前的实现
``` c#
		[Fact]
        public void register_order()
        {
            var containerBuilder = new ContainerBuilder();
            containerBuilder.RegisterType<DbRepository>().As<IRepository>();
            containerBuilder.RegisterType<TestRepository>().As<IRepository>();

            IContainer container = containerBuilder.Build();
            var repository = container.Resolve<IRepository>();
            Assert.Equal(typeof(TestRepository), repository.GetType());
        }
```
如果不想覆盖的话，可以用PreserveExistingDefaults，这样会保留原来注册的实现。
``` c#
		[Fact]
        public void register_order_defaults()
        {
            var containerBuilder = new ContainerBuilder();
            containerBuilder.RegisterType<DbRepository>().As<IRepository>();
            containerBuilder.RegisterType<TestRepository>().As<IRepository>().PreserveExistingDefaults();

            IContainer container = containerBuilder.Build();
            var repository = container.Resolve<IRepository>();
            Assert.Equal(typeof (DbRepository), repository.GetType());
        }
```
可以用Name来区分不同的实现，代替As方法
``` c#
		[Fact]
        public void register_with_name()
        {
            var containerBuilder = new ContainerBuilder();
            containerBuilder.RegisterType<DbRepository>().Named<IRepository>("DB");
            containerBuilder.RegisterType<TestRepository>().Named<IRepository>("Test");

            IContainer container = containerBuilder.Build();
            var dbRepository = container.ResolveNamed<IRepository>("DB");
            var testRepository = container.ResolveNamed<IRepository>("Test");
            Assert.Equal(typeof(DbRepository), dbRepository.GetType());
            Assert.Equal(typeof(TestRepository), testRepository.GetType());
        }
```
如果一个类有多个构造函数的话，可以在注册时候选择不同的构造函数
``` c#
		[Fact]
        public void choose_constructors()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<MyParameter>();
            builder.RegisterType<MyClass>().UsingConstructor(typeof (MyParameter));
            IContainer container = builder.Build();
            var myClass = container.Resolve<MyClass>();
            Assert.NotNull(myClass);
        }
```
AutoFac可以注册一个Assemble下所有的类，当然，也可以根据类型进行筛选
``` c#
		[Fact]
        public void register_assembly()
        {
            var builder = new ContainerBuilder();
            builder.RegisterAssemblyTypes(Assembly.GetExecutingAssembly()).
                Where(t => t.Name.EndsWith("Repository")).
                AsImplementedInterfaces();

            IContainer container = builder.Build();
            var repository = container.Resolve<IRepository>();
            Assert.NotNull(repository);
        }
```