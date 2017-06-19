# Application-Configuration.js - 引导程序和配置文件

AngularJS 有两个执行阶段，

**配置阶段**和**运行阶段**。

_Application-Configuration.js_

会由RequireJS来执行，它会屏蔽掉AngularJS的配置阶段。初始的配置将会使用AngularJS的routeProvider 服务来设定应用程序的路由。在后面浏览示例应用的时候，还会在应用的引导过程中，添加另外的配置函数到配置阶段中去。



