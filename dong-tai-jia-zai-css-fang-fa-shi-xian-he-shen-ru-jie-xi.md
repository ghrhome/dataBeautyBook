# 动态加载css方法实现和深入解析

### 一、方法引用来源和应用

 此动态加载css方法 loadCss，剥离自Sea.js，并做了进一步的优化（优化代码后续会进行分析）。  
 因为公司项目需要用到懒加载来提高网站加载速度，所以将非首屏渲染必需的css文件进行动态加载操作。

### 二、优化后的完整代码

    /*
    * @function 动态加载css文件
    * @param {string} options.url -- css资源路径
    * @param {function} options.callback -- 加载后回调函数
    * @param {string} options.id -- link标签id
    */
    function loadCss(options){
        var url = options.url,
            callback = typeof options.callback == "function" ? options.callback : function(){},
            id = options.id,
            node = document.createElement("link"),
            supportOnload = "onload" in node,
            isOldWebKit = +navigator.userAgent.replace(/.*(?:AppleWebKit|AndroidWebKit)\/?(\d+).*/i, "$1") < 536, // webkit旧内核做特殊处理
            protectNum = 300000; // 阈值10分钟，一秒钟执行pollCss 500次

        node.rel = "stylesheet";
        node.type = "text/css";
        node.href = url;
        if( typeof id !== "undefined" ){
            node.id = id;
        }
        document.getElementsByTagName("head")[0].appendChild(node);

        // for Old WebKit and Old Firefox
        if (isOldWebKit || !supportOnload) {
            // Begin after node insertion
            setTimeout(function() {
                pollCss(node, callback, 0);
            }, 1);
            return;
        }

        if(supportOnload){
            node.onload = onload;
            node.onerror = function() {
                // 加载失败(404)
                onload();
            }
        }else{
            node.onreadystatechange = function() {
                if (/loaded|complete/.test(node.readyState)) {
                    onload();
                }
            }
        }

        function onload() {
            // 确保只跑一次下载操作
            node.onload = node.onerror = node.onreadystatechange = null;

            // 清空node引用，在低版本IE，不清除会造成内存泄露
            node = null;

            callback();
        }

        // 循环判断css是否已加载成功
        /*
        * @param node -- link节点
        * @param callback -- 回调函数
        * @param step -- 计步器，避免无限循环
        */
        function pollCss(node, callback, step){
            var sheet = node.sheet,
                isLoaded;

            step += 1;

            // 保护，大于10分钟，则不再轮询
            if(step > protectNum){
                isLoaded = true;

                // 清空node引用
                node = null;

                callback();
                return;
            }

            if(isOldWebKit){
                // for WebKit < 536
                if(sheet){
                    isLoaded = true;
                }
            }else if(sheet){
                // for Firefox < 9.0
                try{
                    if(sheet.cssRules){
                        isLoaded = true;
                    }
                }catch(ex){
                    // 火狐特殊版本，通过特定值获知是否下载成功
                    // The value of `ex.name` is changed from "NS_ERROR_DOM_SECURITY_ERR"
                    // to "SecurityError" since Firefox 13.0. But Firefox is less than 9.0
                    // in here, So it is ok to just rely on "NS_ERROR_DOM_SECURITY_ERR"
                    if(ex.name === "NS_ERROR_DOM_SECURITY_ERR"){
                        isLoaded = true;
                    }
                }
            }

            setTimeout(function() {
                if(isLoaded){
                    // 延迟20ms是为了给下载的样式留够渲染的时间
                    callback();
                }else{
                    pollCss(node, callback, step);
                }
            }, 20);
        }
    }

### 三、解析代码

##### 一、参数

 本方法支持三个参数，可进行扩展。

**1.1 opations.url**

 url是需要引入的css资源路径，也即&lt;link&gt;标签的href属性内容。

**1.2 options.id**

 id是&lt;link&gt;标签的id属性。这个参数为非必要参数，可不传。主要作用是标记当前&lt;link&gt;标签，方便js进行查找，以确定是否已加载某个css文件。

**1.3 options.callback**

 callback是css文件加载完成后会调用的回调函数。也存在特殊场景下，文件加载失败，回调函数仍旧执行的情况。

##### 二、生成&lt;link&gt;标签，并插入头部head，进行加载资源

```
var url = options.url,
    callback = typeof options.callback == "function" ? options.callback : function(){},
    id = options.id,
    node = document.createElement("link");

node.rel = "stylesheet";
node.type = "text/css";
node.href = url;
if( typeof id !== "undefined" ){
    node.id = id;
}
document.getElementsByTagName("head")[0].appendChild(node);
```

生成一个dom节点&lt;link&gt;，然后配置好rel、type、href等必需的属性值，以便浏览器能正常解析链接的资源。  
 接着，查找到head节点，将&lt;link&gt;节点插入。

##### 三、实现css资源下载状态监控的pollCss方法

 pollCss方法的职责是判断插入的link节点，也即node变量反馈资源是否已加载完成。  
**3.1 判断的主要依据**  
 浏览器加载css资源，会给该link节点生成sheet属性，可以根据浏览器不同，读取sheet属性相关内容，来判断是否已经加载完成。所以第一句语句`var sheet = node.sheet`首先要做的就是获取sheet属性值。  
**3.2 普通浏览器判断**

    try{
        if(sheet.cssRules){
            isLoaded = true;
        }
    }catch(ex){
        // 火狐特殊版本，通过特定值获知是否下载成功
        // The value of `ex.name` is changed from "NS_ERROR_DOM_SECURITY_ERR"
        // to "SecurityError" since Firefox 13.0. But Firefox is less than 9.0
        // in here, So it is ok to just rely on "NS_ERROR_DOM_SECURITY_ERR"
        if(ex.name === "NS_ERROR_DOM_SECURITY_ERR"){
            isLoaded = true;
        }
    }

如果读取sheet.cssRules有值，证明css资源已经链接进页面，并开始解析。此时可以判断资源加载成功。  
 如果读取失败，则根据抛错内容，判断是否有特定name属性`ex.name === "NS_ERROR_DOM_SECURITY_ERR"`。存在，则代表是低版本火狐（9.0以前），且资源已经加载成功。

**3.3 旧webkit内核浏览器判断**

```
var isOldWebKit = +navigator.userAgent.replace(/.*(?:AppleWebKit|AndroidWebKit)\/?(\d+).*/i, "$1") < 536; // webkit旧内核做特殊处理

if(isOldWebKit){
    // for WebKit < 536
    if(sheet){
        isLoaded = true;
    }
}

```

如果是webkit旧内核浏览器，则只需要判断sheet属性值存在，则代表资源加载完成。

**3.4 增加多次循环检测**

```
setTimeout(function() {
    if(isLoaded){
        // 延迟20ms是为了给下载的样式留够渲染的时间
        callback();
    }else{
        pollCss(node, callback, step);
    }
}, 20);
```

触发pollCss方法后，可能第一次检测sheet值，会检测不到。也就代表还没加载完成。所以需要进行轮询。这里是隔20ms进行一次问询，直到资源加载完成为止。

**3.5 轮询容错（针对Sea.js源码的优化）**  
 css资源加载也有可能出错的时机存在，而且存在不触发onerror方法的可能性。如果不加一个保护，则轮询可能一直持续下去，所以需要有一个极限阈值。

```
var protectNum = 300000, // 阈值10分钟，一秒钟执行pollCss 500次
    step = 0;

// 很多代码....

step += 1;

// 保护，大于10分钟，则不再轮询
if(step > protectNum){
    isLoaded = true;

    // 清空node引用
    node = null;

    callback();
    return;
}
```

这里的阈值是轮询10分钟，如果10分钟后，仍然不符合条件，则默认资源已下载完成，执行callback方法，并清空node引用。

##### 四、确定触发pollCss检查的时机

**4.1 pollCss轮询的应用场景**  
 当浏览器内核是旧的webkit内核时，或者不支持&lt;link&gt;节点触发onload方法时，才使用pollCss进行轮询。

  






