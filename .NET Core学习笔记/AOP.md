# AOP 面向切面编程
1. AOP就是在不破坏代码封装的情况下去扩展功能。
- 聚焦业务逻辑
- 代码复用，集中管理
2. Filter,三种注册方式：action注册，控制器注册，全局注册。（类似特性的用法）
    全局注册：
    ```
    services.AddControllersWithViews(
                options =>
                {
                    options.Filters.Add(typeof(CustomerExceptionFilterAttribute));
                }
                );
    ```
3. 特性的依赖注入
Filter需要某个服务，怎么去获取？
特性是编译时确定的，构造函数只能接受常量参数

4. Filter的四种注册方式
- 全局注册
- ServiceFilter 注册
`[ServiceFilter(typeof(CustomerExceptionFilterAttribute))]`
需要在配置类中service注册一下对应的Filter 
`services.AddTransient<CustomerExceptionFilterAttribute>();`
- TypeFilter `[TypeFilter(typeof(CustomerExceptionFilterAttribute))]`
- IFilterFactory 
需要在配置类中service注册一下对应的Filter 
特性用法`[CutomerExceptionFilterFactory(typeof(CustomerExceptionFilterAttribute))]`
```
 public class CutomerExceptionFilterFactoryAttribute : Attribute,IFilterFactory
    {
        private Type _filterType;
        public CutomerExceptionFilterFactoryAttribute(Type type)
        {
            this._filterType = type;
        }
        public bool IsReusable => true;

        /// <summary>
        /// 
        /// </summary>
        /// <param name="serviceProvider">容器</param>
        /// IFilterMetadata是一个空接口，标识是个Filter
        /// <returns></returns>
        public IFilterMetadata CreateInstance(IServiceProvider serviceProvider)
        {
            return (IFilterMetadata)serviceProvider.GetService(_filterType);
        }
    }
```
5. AOP的方式
- Filter:MVC流程，流程外处理不了（404）
- 中间件：任何请求最外层到达，提前做筛选过滤的工作，但中间件没有控制器的相应信息，不适合做业务逻辑控制。
- autofac:可以深入到业务逻辑层做扩展

6. Filter有五大类
- Authorization Filter
- Resource Filter
发生在控制器实例化之前
适合做缓存
- Action Filter
发生在动作执行前后
- Exception Filter
- Result Filter
发生在 视图替换环节

7. 用AOP做缓存
- 使用Resource Filter做缓存
可以避免控制器实例化和Action执行，但是视图会重新执行。因为`context.Result`是一个`ActionResult`，会执行视图
- 业务层缓存
使用IOC进行缓存

- ResponseCache
在请求响应时，添加了一个responseheader,来指导浏览器缓存结果

- 中间件缓存
结合ResponseCache，在中间件完成拦截，可以完全不进入MVC
`services.AddResponseCaching();`
`app.UseResponseCaching();`
`[ResponseCache(Duration = 600)] // 配合服务端缓存`