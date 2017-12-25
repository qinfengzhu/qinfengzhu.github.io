---
layout: post
title: "IOC之WebMvc与WebApi"
date: 2017-12-25   
tag: IOC
---

### Castle.Windsor在Web.Mvc中使用

* 1.建立Controller构造器工厂

```csharp
    public class WindsorControllerFactory:DefaultControllerFactory
    {
        private readonly IKernel kernel;

        public WindsorControllerFactory(IKernel kernel)
        {
            this.kernel = kernel;
        }

        public override void ReleaseController(IController controller)
        {
            kernel.ReleaseComponent(controller);
        }
        protected override IController GetControllerInstance(RequestContext requestContext, Type controllerType)
        {
            if (controllerType == null)
            {
                throw new HttpException(404, string.Format("The controller for path '{0}' could not be found.", requestContext.HttpContext.Request.Path));
            }
            return (IController)kernel.Resolve(controllerType);
        }
    }
```

* 2.构造Controller的Installer

```csharp
    public class ControllerInstaller:IWindsorInstaller
    {
        public void Install(IWindsorContainer container,IConfigurationStore store)
        {
            container.Register(Classes.FromThisAssembly().BasedOn<IController>().LifestyleTransient());
        }
    }
```

* 3.构建容器，安装installers,设置mvc Controller的默认构建工厂

```chsarp
//如下内容在 `Global.asax.cs` 文件中
private static IWindsorContainer container;
protected void Application_Start()
{
        AreaRegistration.RegisterAllAreas();
        RouteConfig.RegisterRoutes(RouteTable.Routes);

        //初始化容器,并且自动安装Installers
        container = new WindsorContainer()
        .Install(FromAssembly.This());
        //这里还可以放置七七八八的注册方式等

        //实例化Mvc-Controller工厂
        var controllerFactory = new WindsorControllerFactory(container.Kernel);

        //设置默认的控制器构造
        ControllerBuilder.Current.SetControllerFactory(controllerFactory);
}
protected void Application_End()
{
	//容器释放
	container.Dispose();
}
```

### Castle.Windsor在Web.Api中使用

* 1.建立新的Windsor服务解析替代默认的Webapi服务解析

```csharp
    public class WindsorDependencyResolver:System.Web.Http.Dependencies.IDependencyResolver
    {
        private readonly IWindsorContainer container;
        public WindsorDependencyResolver(IWindsorContainer container)
        {
            this.container = container;
        }
        public IDependencyScope BeginScope()
        {
            return new DependencyScope(container.Kernel);
        }

        public object GetService(Type serviceType)
        {
            return container.Resolve(serviceType);
        }

        public IEnumerable<object> GetServices(Type serviceType)
        {
            return container.ResolveAll(serviceType).Cast<object>();
        }

        public void Dispose()
        {
            container.Dispose();
        }
    }

    //这里需要用到Ikernel的扩展函数, using Castle.MicroKernel.Lifestyle;
    public class DependencyScope:IDependencyScope
    {
        private readonly IKernel kernel;
        private readonly IDisposable disposable;
        public DependencyScope(IKernel kernel)
        {
            this.kernel = kernel;
            disposable=kernel.BeginScope();
        }
        public object GetService(Type serviceType)
        {
            return kernel.Resolve(serviceType);
        }

        public IEnumerable<object> GetServices(Type serviceType)
        {
            return kernel.ResolveAll(serviceType).Cast<object>();
        }

        public void Dispose()
        {
            disposable.Dispose();
        }
    }
```

* 2.创建WepApi的WindsorInstaller

```csharp
    public class WebApiInstaller:IWindsorInstaller
    {
        public void Install(Castle.Windsor.IWindsorContainer container, Castle.MicroKernel.SubSystems.Configuration.IConfigurationStore store)
        {
            container.Register(
                Classes.FromThisAssembly()
                .BasedOn<ApiController>()
                .LifestyleScoped()
                );
        }
    }
```

* 3.初始化Windsor容器,指定WebApi的服务解析

```csharp
//在Global.asax文件中
private static IWindsorContainer container;
protected void Application_Start()
{
    GlobalConfiguration.Configure(WebApiConfig.Register);

    //初始化Windsor容器
    container = new WindsorContainer();
    container.Install(FromAssembly.This());

    //设置WebApi默认的服务解析
    GlobalConfiguration.Configuration.DependencyResolver = new WindsorDependencyResolver(container);
}
protected void Application_End()
{
	container.Dispose();
}
```