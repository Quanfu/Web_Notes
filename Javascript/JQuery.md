
### 问题
当JQuery访问页面时，由于Session失效得到的响应时`302 redirect` ，在Session失效时想给用户一个提示。于是尝试使用如下代码：
```
jQuery.ajax({
        url :'checkData.ajax',
        complete: function(response) {
            if(response.status == 302) {
                window.alert('You have been logged out of the System');
                goToLogout();
            } else if(response.status == 200) {
                var jsonData =  Ext.decode(response.responseText);
                var resultMessage = jsonData.available;
                if(resultMessage != null && resultMessage != undefined && resultMessage.length > 0) {
                    alert("Message:"+resultMessage);                                        
                } else {
                    callback();
                }
            } else {
                alert("There was some error, please try again in some time.");
            }
        }
    });
    
```
**以上代码无法正常工作** 

再次尝试如下代码：

```
statusCode: {
    302: function() {
        window.alert('You have been logged out of the System');
        goToLogout();
    }
},success :function(result) {
    //var jsonData =  Ext.decode(result);
    var resultMessage = result.available;
    if(resultMessage != null && resultMessage != undefined && resultMessage.length > 0) {
        alert("Message:"+resultMessage);
    } else {
        callback();
    }
},
failure : function() {
        alert("There was some error, please try again in some time.");
}
```
**以上代码无法正常工作**

原因分析：

Ajax 实现处理了`redirect`,会一直等到被重定向到的页面

###JQuery 重定向

[How can I make a page redirect using jQuery?](https://stackoverflow.com/questions/503093/how-can-i-make-a-page-redirect-using-jquery?rq=1)

```
// similar behavior as an HTTP redirect
window.location.replace("http://stackoverflow.com");

// similar behavior as clicking on a link
window.location.href = "http://stackoverflow.com";

```
>jQuery is not necessary, and `window.location.replace(...)` will best simulate an HTTP redirect.

>It is **better** than using `window.location.href =`, because `replace()` does not keep the originating page in the session history, meaning the user won't get stuck in a never-ending back-button fiasco. If you want to simulate someone clicking on a link, use location.href. If you want to simulate an HTTP redirect, use `location.replace`.
