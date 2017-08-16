# AngularJS 异常处理

## $exceptionHandler

对于 Angular digest 中未捕获的异常，由 `$exceptionHandler` 处理。

Any uncaught exception in angular expressions is delegated to this service. The default implementation simply delegates to `$log.error` which logs it into the browser console.



In unit tests, if `angular-mocks.`[`js`](http://lib.csdn.net/base/javascript) is loaded, this service is overridden by  
[mock $exceptionHandler](https://docs.angularjs.org/api/ngMock/service/$exceptionHandler) which aids in testing.

See [AngularJS: API: $exceptionHandler](https://docs.angularjs.org/api/ng/service/$exceptionHandler)

Use a [decorator](https://docs.angularjs.org/api/auto/service/%24provide#decorator), at config time using the [`$provide`](https://docs.angularjs.org/api/auto/service/%24provide) service, on the [`$exceptionHandler`](https://docs.angularjs.org/api/ng/service/%24exceptionHandler) service to perform custom actions when exceptions occur.

Why?: Provides a consistent way to handle uncaught[AngularJS](http://lib.csdn.net/base/angularjs)exceptions for development-time or run-time.

Note: Another option is to override the service instead of using a decorator. This is a fine option, but if you want to keep the default behavior and extend it a decorator is recommended.

```

```



