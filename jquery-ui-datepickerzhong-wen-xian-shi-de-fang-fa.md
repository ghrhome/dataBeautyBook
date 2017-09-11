# JQuery UI Datepicker中文显示的方法

> 出自：http://www.aimks.com/method-to-display-the-jquery-ui-datepicker-chinese.html

Query UI Datepicker这个用于日期显示很方便而且提供了多种样式，可以从jQuery UI中选择喜欢的样式和jQuery UI组件随意下载js库。

如果只是使用datepicker那么选择时之选UI Core和Widgets中的Datepicker，然后选择喜欢的主题，选择版本，下载即可。

不过下载的jQuery UI库中是没有中文的，我们可以将如下js代码放到一个js文件中，然后在文件中引用即可：

```
jQuery(function($){ 
     $.datepicker.regional['zh-CN'] = { 
        clearText: '清除', 
        clearStatus: '清除已选日期', 
        closeText: '关闭', 
        closeStatus: '不改变当前选择', 
        prevText: '< 上月', 
        prevStatus: '显示上月', 
        prevBigText: '<<', 
        prevBigStatus: '显示上一年', 
        nextText: '下月>', 
        nextStatus: '显示下月', 
        nextBigText: '>>', 
        nextBigStatus: '显示下一年', 
        currentText: '今天', 
        currentStatus: '显示本月', 
        monthNames: ['一月','二月','三月','四月','五月','六月', '七月','八月','九月','十月','十一月','十二月'], 
        monthNamesShort: ['一月','二月','三月','四月','五月','六月', '七月','八月','九月','十月','十一月','十二月'], 
        monthStatus: '选择月份', 
        yearStatus: '选择年份', 
        weekHeader: '周', 
        weekStatus: '年内周次', 
        dayNames: ['星期日','星期一','星期二','星期三','星期四','星期五','星期六'], 
        dayNamesShort: ['周日','周一','周二','周三','周四','周五','周六'], 
        dayNamesMin: ['日','一','二','三','四','五','六'], 
        dayStatus: '设置 DD 为一周起始', 
        dateStatus: '选择 m月 d日, DD', 
        dateFormat: 'yy-mm-dd', 
        firstDay: 1, 
        initStatus: '请选择日期', 
        isRTL: false}; 
        $.datepicker.setDefaults($.datepicker.regional['zh-CN']); 
    });
```

```
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>jQuery UI 日期选择器（Datepicker） - 其他月份的日期</title>
  <link rel="stylesheet" href="//apps.bdimg.com/libs/jqueryui/1.10.4/css/jquery-ui.min.css">
  <script src="//apps.bdimg.com/libs/jquery/1.10.2/jquery.min.js"></script>
  <script src="//apps.bdimg.com/libs/jqueryui/1.10.4/jquery-ui.min.js"></script>
  <link rel="stylesheet" href="jqueryui/style.css">
  <script>
      jQuery(function($){ 
          $.datepicker.regional['zh-CN'] = {
              clearText: '清除', 
              clearStatus: '清除已选日期', 
              closeText: '关闭', 
              closeStatus: '不改变当前选择', 
              prevText: '< 上月', 
              prevStatus: '显示上月', 
              prevBigText: '<<', 
              prevBigStatus: '显示上一年', 
              nextText: '下月>', 
              nextStatus: '显示下月', 
              nextBigText: '>>', 
              nextBigStatus: '显示下一年', 
              currentText: '今天', 
              currentStatus: '显示本月', 
              monthNames: ['一月','二月','三月','四月','五月','六月', '七月','八月','九月','十月','十一月','十二月'], 
              monthNamesShort: ['一月','二月','三月','四月','五月','六月', '七月','八月','九月','十月','十一月','十二月'], 
              monthStatus: '选择月份', 
              yearStatus: '选择年份', 
              weekHeader: '周', 
              weekStatus: '年内周次', 
              dayNames: ['星期日','星期一','星期二','星期三','星期四','星期五','星期六'], 
              dayNamesShort: ['周日','周一','周二','周三','周四','周五','周六'], 
              dayNamesMin: ['日','一','二','三','四','五','六'], 
              dayStatus: '设置 DD 为一周起始', 
              dateStatus: '选择 m月 d日, DD', 
              dateFormat: 'yy-mm-dd', 
              firstDay: 1, 
              initStatus: '请选择日期', 
              isRTL: false}; 
          $.datepicker.setDefaults($.datepicker.regional['zh-CN']); 
      });
      $(function() {
          $( "#datepicker" ).datepicker({
              showOtherMonths: true,
              selectOtherMonths: true,
              showButtonPanel: true,
              showOn: "both",
              buttonImageOnly: true,
              buttonImage: "calendar.gif",
              buttonText: "",
              changeMonth: true,
              changeYear: true
          });
      });
  </script>
</head>
<body>
 
<p>日期：<input type="text" id="datepicker"></p>
 
 
</body>
</html>
```



