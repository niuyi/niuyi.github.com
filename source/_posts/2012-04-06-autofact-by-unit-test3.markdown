---
layout: post
title: "AutoFac使用方法总结:Part III"
date: 2012-04-06 23:48
comments: true
categories: 
---

<h3>生命周期</h3>
AutoFac中的生命周期概念非常重要，AutoFac也提供了强大的生命周期管理的能力。
<!-- more -->
AutoFac定义了三种生命周期：
	Per Dependency
	Single Instance
	Per Lifetime Scope

Per Dependency为默认的生命周期，也被称为'transient'或'factory'，其实就是每次请求都创建一个新的对象
``` c#
		[Fact]
        public void per_dependency()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<MyClass>().InstancePerDependency();
            IContainer container = builder.Build();
            var myClass1 = container.Resolve<MyClass>();
            var myClass2 = container.Resolve<MyClass>();
            Assert.NotEqual(myClass1,myClass2);
        }
```
Single Instance也很好理解，就是每次都用同一个对象
``` c#
		[Fact]
        public void single_instance()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<MyClass>().SingleInstance();
			
            IContainer container = builder.Build();
            var myClass1 = container.Resolve<MyClass>();
            var myClass2 = container.Resolve<MyClass>();
			
            Assert.Equal(myClass1,myClass2);
        }
```
Per Lifetime Scope，同一个Lifetime生成的对象是同一个实例
``` c#
		[Fact]
        public void per_lifetime_scope()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<MyClass>().InstancePerLifetimeScope();
			
            IContainer container = builder.Build();
            var myClass1 = container.Resolve<MyClass>();
            var myClass2 = container.Resolve<MyClass>();
			
            ILifetimeScope inner = container.BeginLifetimeScope();
            var myClass3 = inner.Resolve<MyClass>();
            var myClass4 = inner.Resolve<MyClass>();
			
            Assert.Equal(myClass1,myClass2);
            Assert.NotEqual(myClass2,myClass3);
            Assert.Equal(myClass3,myClass4);
        }
```
``` c#
		 [Fact]
        public void life_time_and_dispose()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<Disposable>();

            using (IContainer container = builder.Build())
            {
                var outInstance = container.Resolve<Disposable>(new NamedParameter("name", "out"));

                using(var inner = container.BeginLifetimeScope())
                {
                    var inInstance = container.Resolve<Disposable>(new NamedParameter("name", "in"));
                }//inInstance dispose here
            }//out dispose here
        }
```