---
layout: post
title: "AutoFac使用方法总结:Part II"
date: 2012-04-06 23:38
comments: true
categories: 
---
<h3>事件</h3>
AutoFac支持三种事件：OnActivating，OnActivated，OnRelease。OnActivating在注册组件使用之前会被调用，此时可以替换实现类或者进行一些其他的初始化工作，OnActivated在实例化之后会被调用，OnRelease在组件释放之后会被调用。
<!-- more -->
``` c#
		public class MyEvent : IDisposable
		{
			public MyEvent(string input)
			{
				Console.WriteLine(input);
			}

			public MyEvent()
			{
				Console.WriteLine("Init");
			}

			public void Dispose()
			{
				Console.WriteLine("Dispose");
			}
		}
```
``` c#
		public void test_event()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<MyEvent>().
                OnActivating(e => e.ReplaceInstance(new MyEvent("input"))).
                OnActivated(e => Console.WriteLine("OnActivated")).
                OnRelease(e => Console.WriteLine("OnRelease"));

            
            using (IContainer container = builder.Build())
            {
                using (var myEvent = container.Resolve<MyEvent>())
                {
                }
            }
        }
``` 
此时的输出为：
``` 
Init
input
OnActivated
Dispose
OnRelease
``` 
利用事件可以在构造对象之后调用对象的方法：
``` c#
		[Fact]
        public void call_method_when_init()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<MyClassWithMethod>().OnActivating(e => e.Instance.Add(5));
            IContainer container = builder.Build();
            Assert.Equal(5, container.Resolve<MyClassWithMethod>().Index);
        }
		public class MyClassWithMethod
		{
			public int Index { get; set; }
			public void Add(int value)
			{
				Index = Index + value;
			}
		}
```
<h3>循环依赖</h3>
循环依赖是个比较头疼的问题，在AutoFac中很多循环依赖的场景不被支持：
``` c#
		public class ClassA
		{
			private readonly ClassB b;

			public ClassA(ClassB b)
			{
				this.b = b;
			}
		}

		public class ClassB
		{
			public ClassA A { get; set; }
			
		}
		
		[Fact]
        public void circular_dependencies_exception()
        {
            var builder = new ContainerBuilder();
            builder.Register(c => new ClassB(){A = c.Resolve<ClassA>()});
            builder.Register(c => new ClassA(c.Resolve<ClassB>()));
            IContainer container = builder.Build();
            Assert.Throws(typeof(DependencyResolutionException), ()=>container.Resolve<ClassA>());
        }
```
可以部分的解决这种循环依赖的问题，前提是ClassA和ClassB的生命周期不能都是InstancePerDependency
``` c#
		[Fact]
        public void circular_dependencies_ok()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<ClassB>().
                PropertiesAutowired(PropertyWiringFlags.AllowCircularDependencies).SingleInstance();
            builder.Register(c => new ClassA(c.Resolve<ClassB>()));
            IContainer container = builder.Build();
            Assert.NotNull(container.Resolve<ClassA>());
            Assert.NotNull(container.Resolve<ClassB>());
            Assert.NotNull(container.Resolve<ClassB>().A);
        }
```