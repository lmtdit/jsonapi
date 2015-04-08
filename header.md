# header头规范

### 标准头

```
Content-Type: application/vnd.api+json
Accept: application/vnd.{团队名称}.{版本号: v1, v2, ...}+json
```

说明:

- application/vnd.api+json : JSON API 在 IANA 机构完成注册的 MIME 类型
- application/vnd.{团队名称}.{版本号: v1, v2, ...}+json 服务器接收类型

### 认证头

```
Authorization: Bearer {用户token}
```

### 其他需要注意的内容(建议)

```
User-Agent: {你需要的内容}
```

