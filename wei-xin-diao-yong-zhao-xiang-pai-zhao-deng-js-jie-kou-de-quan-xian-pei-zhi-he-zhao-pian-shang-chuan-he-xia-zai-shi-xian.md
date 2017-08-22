# 微信调用照相拍照等 js 接口的权限配置 和 照片上传和下载实现

直接上代码：

1. 前端调试代码：

```
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>调用微信相机</title>
<link rel="stylesheet" href="css/weui.min.css"/>
</head>
<body>
<br/><br/>
<div>
    <div class="hd">
        <h1 class="page_title" ><span id="current_user"></span></h1>
    </div>
        <br/>
    <div class="page slideIn button">
        <div class="bd spacing">
            <br/><br/><br/><br/>
            <a class="weui_btn weui_btn_primary" href="javascript:;" onclick="take_a_photo()">调用微信相机</a>
        </div>
    </div>
</div>
 
<br/><br/><br/><br/>
<div>
      appId:<span id="appId"></span><br/>
      timestamp:<span id="timestamp"></span><br/>
      nonceStr:<span id="nonceStr"></span><br/>
      jsapi_ticket:<span id="jsapi_ticket"></span><br/>
      signature:<span id="signature"></span><br/>
     
      originalStr:<span id="originalStr"></span><br/>
      scan_result:<span id="result"></span><br/>
</div>   
 
<script type='text/javascript' src='js/jquery.min.js'></script>
<script type="text/javascript" src="http://res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>
<script type="text/javascript" src="js/weixin_util.js"></script>
<script type="text/javascript">
// 通过config接口注入权限验证配置
$(function(){
    debugger;
    get_wx_config();
})
 
function get_wx_config(){
    debugger;
    //url（当前网页的URL，不包含#及其后面部分）
    var url = window.location.href.split('#')[0];
    var indata = {"url":url};
    //$.post("http://wwww.axxx.cn/xxx/weixin/getConfigInfo.json", indata, function(data){
    $.post("http://localhost:8080/xxx/weixin/getConfigInfo.json", indata, function(data){
        debugger;
        if(data.rs == 'f'){
            alert("系统错误");
        }else{
            var result = data.body;
            $("#appId").text(result.appId);
            $("#timestamp").text(result.timestamp);
            $("#nonceStr").text(result.nonceStr);
            $("#jsapi_ticket").text(result.jsapi_ticket);
            $("#signature").text(result.signature);
            $("#originalStr").text(result.originalStr);
             
            //步骤三：通过config接口注入权限验证配置
            wx.config({
                debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
                appId: result.appId,   // 必填，公众号的唯一标识
                timestamp: result.timestamp, // 必填，生成签名的时间戳
                nonceStr: result.nonceStr, // 必填，生成签名的随机串
                signature: result.signature,// 必填，签名，见附录1
                jsApiList: ["chooseImage", "previewImage", "uploadImage", "downloadImage"] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
                // 基本思路是，上传图片到微信服务器->下载多媒体接口讲图片下载到服务器->返回服务器存储路径->前台显示
            });
             
            // 步骤四：通过ready接口处理成功验证
            wx.ready(function(){
                alert("wx.config success.");
            });
             
            wx.error(function(res){
                alert("wx.config failed.");
                //alert(res);
                // config信息验证失败会执行error函数，如签名过期导致验证失败，具体错误信息可以打开config的debug模式查看，
                // 也可以在返回的res参数中查看，对于SPA可以在这里更新签名。
            });
             
            // {"errMsg":"config:invalid signature"}
        }
      },'json');
}
 
 
// 图片接口
// 拍照、本地选图
var images = {
    localId: [],
    serverId: []
}; 
 
// 拍照或者选择照片
function take_a_photo(){
    wx.chooseImage({
        count: 1, // 默认9，这里每次只处理一张照片
        sizeType: ['original', 'compressed'],   // 可以指定是原图还是压缩图，默认二者都有
        sourceType: ['album', 'camera'],        // 可以指定来源是相册还是相机，默认二者都有
        success: function (res) {
            images.localId = res.localIds;      // 返回选定照片的本地ID列表，localId可以作为img标签的src属性显示图片
             
            var i = 0, length = images.localId.length; 
            function upload() { 
                wx.uploadImage({ 
                    localId: images.localId[i], 
                    success: function(res) { 
                        i++; 
                        alert('已上传：' + i + '/' + length); 
                        images.serverId.push(res.serverId);
                         
                        // res.serverId 就是 media_id，根据它去微信服务器读取图片数据：
                        var indata = {"media_id":res.serverId};
                        $.post("/weixin/getPhoto.json", indata, function(data){
                            if(data.rs == 'f'){
                                alert("系统错误");
                            }else{
                                alert(data.body);   // 返回 图片在我们自己的服务器的url
                            }
                          },'json');
                         
                        if (i < length) { 
                            upload(); 
                        } 
                    }, 
                    fail: function(res) { 
                        alert(JSON.stringify(res)); 
                    } 
                }); 
            } 
            upload(); 
        } 
    });
}
</script>
</body>
</html>
```

前端这里 

```
var url = window.location.href.split('#')[0];<br>是比较容易犯错的地方。<br><br>
```

2. 后端接口：

```
@NoLogin
    @RequestMapping(value="/getConfigInfo.json", method=RequestMethod.POST)
    @ResponseBody
    public Object getConfigInfo(String url) throws NoSuchAlgorithmException{
        String noncestr = "dsfww";
        JsapiTicket ticket = AccessTokenJsapiTicketThread.getTicket();
         
        logger.debug("ticket::::::;" + JSON.toJSONString(ticket));
        System.out.println("ticket::::::;" + JSON.toJSONString(ticket));
         
        if(ticket != null){
            long timestamp = new Date().getTime();
            StringBuilder sb = new StringBuilder("jsapi_ticket=");
            sb.append(ticket.getTicket()).append("&noncestr=").append(noncestr)
            .append("&timestamp=").append(timestamp).append("&url=").append(url);
             
            MessageDigest md = MessageDigest.getInstance("SHA-1");
            // 对接后的字符串进行sha1加密
            byte[] digest = md.digest(sb.toString().getBytes());
            String signature = SignUtil.byteToStr(digest).toLowerCase();
             
            Map<String, String> map = new HashMap<String, String>();
            map.put("jsapi_ticket", ticket.getTicket());
            map.put("appId", WxPayConfPub.APPID);
            map.put("timestamp", String.valueOf(timestamp));
            map.put("nonceStr", noncestr);
            map.put("signature", signature);
            map.put("originalStr", sb.toString());
            logger.debug(JSON.toJSONString(map));
            System.out.println(JSON.toJSONString(map));
            return JsonConvertor.convertSuccessResult(map);
        }
         
        return JsonConvertor.convertFailResult(ErrorCodeEnum.SYSTEM_ERROR);
    }
     
    @NoLogin
    @RequestMapping(value="/getPhoto.json", method=RequestMethod.POST)
    @ResponseBody
    public Object getPhoto(String media_id) throws NoSuchAlgorithmException{
        //http请求方式: GET,https调用
//        var url = "https://api.weixin.qq.com/cgi-bin/media/get?access_token=ACCESS_TOKEN&media_id=MEDIA_ID";
        AccessToken token = AccessTokenJsapiTicketThread.accessToken;
        String url = "https://api.weixin.qq.com/cgi-bin/media/get?access_token=" + token.getAccess_token() + "&media_id=" + media_id;
        HttpsURLConnection httpsUrl = null;
        InputStream inputStream = null;
        Date now = new Date();
        String saveFileName = null;
        try {
            httpsUrl = HttpUtil.initHttpsConnection(url, "GET");
            int responseCode = httpsUrl.getResponseCode();
            if (responseCode == 200) {
                // 从服务器返回一个输入流
                inputStream = httpsUrl.getInputStream();
                 
                byte[] data = new byte[1024];
                int len = 0;
                FileOutputStream fileOutputStream = null;
                 
                saveFileName = DateUtil.convertYMDH(now) + RandomStringUtils.random(6, true, true) + ".jpg";;
 
                // 绝对路径
                String path = imageRootPath + DateUtil.convertYMD(now) + "/" + saveFileName;
                 
                File dir = new File(imageRootPath + DateUtil.convertYMD(now) + "/");
                if (!dir.exists()) {
                    FileUtils.forceMkdir(dir);
                }
                 
                try {
                    fileOutputStream = new FileOutputStream(path);
                    while ((len = inputStream.read(data)) != -1) {
                        fileOutputStream.write(data, 0, len);
                    }
                    fileOutputStream.flush();
                } catch (IOException e) {
                  e.printStackTrace();
                } finally {
                     if (inputStream != null) {
                        try {
                          inputStream.close();
                        } catch (IOException e) {
                        }
                     }
                      if (fileOutputStream != null) {
                        try {
                          fileOutputStream.close();
                        } catch (IOException e) {
                        }
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 返回图片路径
        return JsonConvertor.convertSuccessResult(DateUtil.convertYMD(now) + "/" + saveFileName);
    }
```

/getConfigInfo.json 接口是配置 jsapi 权限认证。

/getPhoto.json 是从 微信服务器下载照片，保存到我们自己的服务器上。然后将我们自己服务器的地方返回给前端使用。







