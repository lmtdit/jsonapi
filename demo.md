# 前后端的API封装样例

本文定义了一个标准的 Web 前后端 Ajax JSON 接口规范，并提供了基于 jQuery 的前端实现与 PHP 后端实现。

对于返回的 Ajax 数据格式，建议采用如下 JSON 格式，一般如下：

```javascript
{"code": code-value, "message": message-value, "data": data-value}
```

对于以上数据格式的解释如下：

+ ```code-value```：必须，整型，调用所返回的状态码，其值必须大于等于零。```0``` 值，固定用来表示调用正常；大于 ```0``` 的值，可以由所调用的后台来定义，表示各种出错状态；
+ ```message-value```：可选，字符串，对 ```code-value``` 做进一步描述。当 ```code-value``` 为 ```0``` 值时，“```message``` 值对”可选。当 ```code-value``` 为大于 ```0``` 的值时，“```message``` 值对”必填。
+ ```data``` 及 ```data-value```：可选，用来存储返回的实体数据。如果 ```data``` 存在，```data-value``` 可为常规数据类型或复合数据类型。如果其为复合数据类型，则为“值对”数据结构，或多维“值对”数据结构。但其中必须包含 ```id``` 属性标识的值对，一般来说可选的其他属性包括（```text```，```name``` 等）。示例如下：

	+ 单一值对

	```javascript
	{"id": 1, "text": "user1", <自定义值对 1>, <自定义值对 2> ...}
	```

	+ 多维值对

	```javascript
	[
	    {"id": 1, "text": "user1", <自定义值对 1>, <自定义值对 2> ...},
	    {"id": 2, "text": "user2", <自定义值对 1>, <自定义值对 2> ...}
	]
	```

	+ 或

	```javascript
	{
	    "list": [
	        {"id": 1, "text": "user1", <自定义值对 1>, <自定义值对 2> ...},
	        {"id": 2, "text": "user2", <自定义值对 1>, <自定义值对 2> ...}
	    ],
	    "pageIndex": 1,
	    "pageCount": 10,
	    "hasPrev": false,
	    "hasNext": true
	}
	```

## jQuery 对API的接收封装

由于以上已经对数据格式做出了定义，下面对 jsonAPI 的调用方法做出封装，以适用此种建议格式。以下的实现基于 jQuery.ajax 方法，你可以参考此实现，封装出适用于自己的 jsonAPI 调用函数。

```javascript
;(function($) {
if ($.jsonAPI) {
	return;
}

function isString(obj) {
	return typeof obj == "string" || Object.prototype.toString.call(obj) === "[object String]";
}

/*
 * jQuery.jsonAPI:  Wrap "jQuery.ajax" method with JSON response style.
 *               Support all ajax parameters as default. Extending or Changing below parameters' usage.
 * @dataType:    Optional. The value is "JSON" forever. You can't override it.
 * @traditional: Optional. Defaut value is "true". Same explanation as "jQuery.ajax" method.
 * @errorCodes:  Optional. Extended property, which used for list error codes you want to catching.
 *               Codes list will separate by comma, or use "*" as the wildcard.
 *               The default HTTP error code is -1. So, you can catch HTTP error also.
 *               Example:
 *                 errorCodes: "2,3,4", or errorCodes: "*"
 * @success:     Optional. Callback for the Ajax response, when "code" value is "0".
 * @error:       Optional. Callback for the Ajax response, when "code" value other than "0".
 */
$.jsonAPI = function(settings) {

	// It's used for handling "success" and "error" callback.
	function callback(result) {
		if (!$.isPlainObject(result) || !$.isNumeric(result["code"]) ||
			(result["code"] != 0 && !result["message"]) ||
			(result["code"] != 0 && !isString(result["message"]))) {
			result = {
				code: -6,
				message: "Network response is illegal."
			};
		}

		if (result.code == 0) {
			if ($.isFunction(settings.success)) {
				settings.success(result);
			}
		}
		else if (settings.errorCodes == "*" ||
				(settings.errorCodes && new RegExp( "^" + result.code + "$|" +
											"^" + result.code + "[\s,]+|" +
											"[\s,]+" + result.code + "$|" +
											"[\s,]+" + result.code + "[\s,]+", "i").
											test(settings.errorCodes + ""))) {
			if ($.isFunction(settings.error)) {
				settings.error(result);
			}
		}
		else {
			// Below alert popup maybe not good for your project.
			// So, you can change the alert to any other warning way.
			alert(result.message);
		}
	}

	var options = $.extend({traditional: true}, settings, {
		dataType: "json",
		success: callback,
		error: function(xhr, status) {
			var map = {
				"abort": {
					code: -1,
					message: "Network request is aborted."
				},
				"parsererror": {
					code: -2,
					message: "Network response is parsing error."
				},
				"timeout": {
					code: -3,
					message: "Network request is timeout."
				},
				"error": {
					code: -4,
					message: "Network error."
				}
			};

			var result = map[status];
			if (!result) {
				result = {
					code: -5,
					message: "Unknown network error."
				}
			}

			// You can change the default HTTP error code if it conflict.
			callback(result);
		}
	});

	return $.ajax(options);
}
})(jQuery);
```


## API的PHP输出实现
在这里只提供出一个 PHP 版本的对应实现，供后端参考，使用其他后端语言开发的同学，可做自己的实现。

```php
<?php
    /**
     * PHP version response method for jQuery.jsonAPI
     */
    function jsonAPIOut($code = 0, $message = "", $data = "") {
        header("Content-Type: text/javascript; charset=utf-8");
        header("Expires: Mon, 26 Jul 1997 05:00:00 GMT");
        header("Last-Modified: " . gmdate("D, d M Y H:i:s") . " GMT");
        header("Cache-Control: no-store, no-cache, must-revalidate");
        header("Cache-Control: post-check=0, pre-check=0", false);
        header("Pragma: no-cache");

        // code field for json output
        $out = array('code' => $code);
        if ($code < 0) {
            $out['message'] = "jsonAPIOut Error: the 'code' parameter should not less than 0.";
            echo json_encode($out);
            return;
        }

        // message field for json output
        if ($message !== "") {
            $out['message'] = $message;
        }
        else if ($code > 0 && $message === "") {
            $out['message'] = "jsonAPIOut Error: the 'message' parameter should define if the 'code' parameter great than 0." ;
            echo json_encode($out);
            return;
        }

        // data field for json output
        if ($data !== "") {
            $out['data'] = $data;
        }

        // output the json structure as string.
        echo json_encode($out);
    }
?>
```