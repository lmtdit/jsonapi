# API错误描述样例

```javascript
{
   "message": "Validation Failed",
   "errors": [
     {
       "resource": "Issue",
       "field": "title",
       "code": "missing_field"
     }
   ]
 }
```

所有错误包括下面信息

- missing: 说明某个字段的值代表的资源不存在
- invalid: 某个字段的值非法，接口文档中会提供相应的信息
- missing_field: 缺失某个必须的字段
- already_exist: 发送的资源中的某个字段的值和服务器中已有的某个资源冲突，常见于某些值全局唯一的字段，比如 @ 用的用户名（这个错误我有纠结，因为其实有 409 状态码可以表示，但是在修改某个资源时，很一般显然请求中不止是一种错误，如果是 409 的话，多种错误的场景就不合适了）
