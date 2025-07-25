# AdventureX 指纹规则编写指南

## 📋 目录

- [什么是指纹规则](#什么是指纹规则)
- [规则文件格式](#规则文件格式)
- [DSL表达式语法](#dsl表达式语法)
- [基础函数详解](#基础函数详解)
- [高级功能](#高级功能)
- [实战案例](#实战案例)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)

## 什么是指纹规则

指纹规则是用来识别网站技术栈的配置文件。AdventureX通过分析HTTP响应的内容（如响应体、响应头、状态码等），使用DSL（领域特定语言）表达式来匹配特定的技术特征，从而识别目标网站使用的技术。

**简单理解**：就像人的指纹一样，每种技术都有自己独特的"指纹"特征，我们通过编写规则来识别这些特征。

## 规则文件格式

指纹规则使用YAML格式编写，基本结构如下：

```yaml
技术名称:
  dsl:
    - "DSL表达式1"
    - "DSL表达式2"
```

### 基础规则示例

```yaml
# 识别Apache服务器
apache:
  dsl:
    - "contains(header, 'Apache')"

# 识别WordPress
wordpress:
  dsl:
    - "contains(body, 'wp-content')"
    - "contains(body, 'wp-includes')"
```

### 完整规则示例

```yaml
# 完整的规则配置
nginx:
  path: "/nginx_status" # 可选：主动探测路径
  condition: or        # 可选：匹配条件（or/and）
  dsl:
    - "contains(header, 'nginx')"
    - "server('nginx')"
  tags: nginx # 可选：技术分类
```

## DSL表达式语法

DSL（Domain Specific Language）是AdventureX用来描述匹配规则的表达式语言。

### 基本语法规则

1. **字符串用引号包围**：`"text"` 或 `'text'`
2. **函数调用格式**：`function(参数1, 参数2)`
3. **逻辑运算符**：`&&`（且）、`||`（或）
4. **比较运算符**：`==`、`!=`、`>`、`<`、`>=`、`<=`

### 语法示例

```yaml
simple-rule:
  dsl:
    - "contains(body, 'WordPress')"

complex-rule:
  dsl:
    - "contains(body, 'admin') && status_code == 200"
    - "server('Apache') || contains(header, 'nginx')"
```

## 基础函数详解

### 1. contains() - 内容包含检查

**用法**：`contains(source, "text")`

**支持的source**：

- `body` - 响应体内容
- `header` - 响应头内容
- `title` - 页面标题
- `server` - 服务器头信息

```yaml
# 检查响应体是否包含特定文本
wordpress:
  dsl:
    - "contains(body, 'wp-content')"

# 检查响应头是否包含特定内容
apache:
  dsl:
    - "contains(header, 'Apache')"

# 检查页面标题
admin-panel:
  dsl:
    - "contains(title, '管理后台')"
```

### 2. regex() - 正则表达式匹配

**用法**：`regex(source, "pattern")` 或 `regex("pattern")`（默认在body中搜索）

```yaml
# 匹配版本号
php:
  dsl:
    - "regex(header, 'PHP/\\d+\\.\\d+\\.\\d+')"

# 在响应体中匹配邮箱格式
email-found:
  dsl:
    - "regex('[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}')"

# 匹配特定格式的错误信息
sql-server:
  dsl:
    - "regex(body, 'Microsoft.*SQL Server.*Error')"
```

### 3. status_code - 状态码检查

**用法**：`status_code 运算符 数值`

**支持的运算符**：`==`、`!=`、`>`、`<`、`>=`、`<=`

```yaml
# 检查特定状态码
forbidden-access:
  dsl:
    - "status_code == 403"

# 检查成功状态码范围
success-response:
  dsl:
    - "status_code >= 200 && status_code < 300"

# 排除特定状态码
not-found-page:
  dsl:
    - "status_code != 404"
```

### 4. title() - 页面标题检查

**用法**：`title("text")`

```yaml
# 检查页面标题
admin-login:
  dsl:
    - "title('Admin Login')"

# 中文标题检查
chinese-admin:
  dsl:
    - "title('后台管理系统')"
```

### 5. server() - 服务器头检查

**用法**：`server("text")`

```yaml
# 检查服务器类型
nginx:
  dsl:
    - "server('nginx')"

iis:
  dsl:
    - "server('Microsoft-IIS')"
```

### 6. header() - 自定义响应头检查

**用法**：

- `header("header-name")` - 检查响应头是否存在
- `header("header-name", "value")` - 检查响应头是否包含特定值

```yaml
# 检查特定响应头
cors-enabled:
  dsl:
    - "header('Access-Control-Allow-Origin')"

# 检查响应头的值
security-header:
  dsl:
    - "header('X-Frame-Options', 'DENY')"

# 检查自定义头信息
custom-app:
  dsl:
    - "header('X-Powered-By', 'Express')"
```

### 7. contains_all() - 多文本包含检查

**用法**：`contains_all('text1', 'text2', 'text3')`

检查响应体是否**同时包含**所有指定的文本。

```yaml
# 必须同时包含多个关键字
laravel:
  dsl:
    - "contains_all('Laravel', 'Illuminate', 'framework')"

# 检查登录页面的多个元素
login-page:
  dsl:
    - "contains_all('username', 'password', 'login', 'submit')"
```

### 8. icon() - 图标哈希匹配

**用法**：`icon('path', 'hash')`

通过计算图标文件的MD5哈希值来精确识别技术。

```yaml
# WordPress默认图标
wordpress:
  dsl:
    - "icon('/favicon.ico', 'a5bfbceb1e6230b9cbbc8e5dd2314be9')"

# 自定义应用图标
custom-app:
  dsl:
    - "icon('/assets/favicon.png', '6d07440dcda38480ac6fd8c32edf0102')"
```

## 高级功能

### 1. 逻辑条件控制

**condition字段**用于控制多个DSL表达式之间的逻辑关系：

- `or`（默认）：任意一个表达式匹配即可
- `and`：所有表达式都必须匹配

```yaml
# OR条件（默认）- 任意匹配即可
apache:
  # condition: or  # 可以省略，默认就是or
  dsl:
    - "contains(header, 'Apache')"
    - "server('Apache')"
    - "contains(body, 'Apache Server')"

# AND条件 - 必须全部匹配
secure-admin:
  condition: and
  dsl:
    - "contains(title, 'Admin')"
    - "status_code == 200"
    - "contains(header, 'Set-Cookie')"
```

### 2. 主动探测功能

**path字段**用于主动探测特定路径，而不是被动等待流量：

```yaml
# 主动探测robots.txt
website-info:
  path: "/robots.txt"
  dsl:
    - "contains(body, 'Disallow')"

# 主动探测管理后台
admin-panel:
  path: "/admin"
  dsl:
    - "status_code == 200"
    - "contains(body, 'login')"

# 主动探测API接口
api-server:
  path: "/api/version"
  dsl:
    - "contains(body, 'version')"
    - "contains(header, 'application/json')"
```

### 3. 复合逻辑表达式

在单个DSL表达式中使用逻辑运算符：

```yaml
# 复合条件
web-application:
  dsl:
    - "contains(body, 'login') && status_code == 200"
    - "server('nginx') || server('Apache')"
    - "contains(header, 'PHP') && contains(body, 'mysql')"

# 复杂的版本检测
php-version:
  dsl:
    - "regex(header, 'PHP/[5-7]\\.[0-9]+') && !contains(header, 'PHP/5.0')"
```

## 实战案例

### 案例1：识别WordPress网站

```yaml
wordpress:
  dsl:
    - "contains(body, 'wp-content')"
    - "contains(body, 'wp-includes')"
    - "contains(body, 'wp-admin')"
    - "contains(header, 'wordpress')"
  condition: or  # 任意一个匹配即可
  tags: wordpress
```

### 案例2：识别管理后台

```yaml
admin-panel:
  condition: and  # 必须同时满足
  dsl:
    - "contains(title, 'admin')"
    - "status_code == 200"
    - "contains(body, 'username')"
    - "contains(body, 'password')"
   tags: admin-page
```

### 案例3：识别特定框架版本

```yaml
spring-boot:
  dsl:
    - "contains(header, 'Spring')"
    - "regex(body, 'spring-boot-starter-[a-z]+')"
    - "contains(body, 'Whitelabel Error Page')"
  condition: or
  tags: java
```

### 案例4：主动探测API

```yaml
api-documentation:
  path: "/api/docs"
  dsl:
    - "status_code == 200"
    - "contains(body, 'swagger')"
    - "contains(header, 'application/json')"
  condition: and
  tags: api-doc,infoleak,swagger
```

### 案例5：复杂的数据库识别

```yaml
mysql-database:
  dsl:
    - "regex(body, 'MySQL.*Error|mysql.*error')"
    - "contains(body, 'mysql_connect')"
    - "regex(header, 'MySQL/\\d+\\.\\d+')"
  condition: or
  tags: mysql,database
  
# 更精确的版本识别
mysql-5x:
  condition: or
  tags: mysql,mysql5
  dsl:
    - "regex(body, 'MySQL 5\\.[0-9]+\\.[0-9]+')"
    - "contains(header, 'MySQL/5.')"

```

### 案例6：中文应用识别

```yaml
seeyon-oa:
  path: "/seeyon/main.do"
  dsl:
    - "contains(body, '致远协同')"
    - "contains(body, 'seeyon')"
  condition: or
  tags: oa

discuz:
  dsl:
    - "contains(body, 'Powered by Discuz')"
    - "contains(body, 'discuz_uid')"
    - "contains(title, 'Discuz')"
  condition: or
  tags: bbs,forum
```

## 最佳实践

### 1. 规则命名规范

```yaml
# ✅ 好的命名
apache-httpd:
wordpress-cms:
nginx-proxy:
mysql-database:

# ❌ 避免的命名
web:
app:
test:
```

### 2. 选择合适的匹配条件

```yaml
# 宽松匹配 - 用于识别大类技术
web-server:
  condition: or  # 任意匹配
  dsl:
    - "server('Apache')"
    - "server('nginx')"
    - "server('IIS')"

# 严格匹配 - 用于精确识别
specific-version:
  condition: and  # 全部匹配
  dsl:
    - "contains(body, 'WordPress')"
    - "regex(body, 'Version 5\\.[0-9]+')"
    - "status_code == 200"
```

### 3. 合理使用主动探测

```yaml
# ✅ 适合主动探测的场景
robots-check:
  path: "/robots.txt"
  dsl:
    - "contains(body, 'Disallow')"

version-info:
  path: "/version.json"
  dsl:
    - "contains(body, 'version')"

# ❌ 不建议的主动探测
homepage:
  path: "/"  # 首页通常已被被动扫描覆盖
  dsl:
    - "status_code == 200"
```

### 4. 性能优化建议

```yaml
# ✅ 高效的规则
quick-check:
  dsl:
    - "contains(header, 'Server: Apache')"  # 检查响应头比检查body更快

# ✅ 具体的匹配
specific-match:
  dsl:
    - "contains(body, 'wp-content/themes')"  # 具体的字符串匹配

# ❌ 性能较差的规则
slow-check:
  dsl:
    - "regex(body, '.*wordpress.*')"  # 过于宽泛的正则表达式
```

### 5. 规则测试建议

```yaml
# 添加tags便于管理
wordpress:
  dsl:
    - "contains(body, 'wp-content')"
  tags: cms

# 使用描述性的DSL表达式
login-page:
  dsl:
    - "contains(title, 'login') && status_code == 200"  # 清晰的逻辑
  tags: login-page
```

## 常见问题

### Q1: 如何调试规则是否正确？

**A**: 使用AdventureX的日志功能，观察匹配结果：

```bash
# 启用调试模式
./advent -m finger -u target.com --debug

# 观察匹配日志
INFO fingerprint: 🎯 http://target.com <rule-name> <matched-dsl>
```

### Q2: 为什么我的规则没有匹配？

**A**: 常见原因：

1. **大小写敏感**：DSL匹配是不区分大小写的，但要确保字符串完全正确
2. **特殊字符**：正则表达式中的特殊字符需要转义
3. **条件设置**：检查`condition`字段设置是否正确

```yaml
# ❌ 可能有问题的规则
wrong-rule:
  condition: and  # 要求所有条件都满足
  dsl:
    - "contains(body, 'WordPress')"
    - "contains(body, 'Drupal')"  # 不可能同时是WordPress和Drupal

# ✅ 修正后的规则
cms-detection:
  condition: or  # 任意一个匹配即可
  dsl:
    - "contains(body, 'WordPress')"
    - "contains(body, 'Drupal')"
```

### Q3: 如何优化规则性能？

**A**: 性能优化技巧：

1. **优先检查响应头**：比检查响应体更快
2. **使用具体的字符串**：避免过于宽泛的正则表达式
3. **合理设置条件**：避免不必要的`and`条件

```yaml
# ✅ 高性能规则
fast-rule:
  dsl:
    - "server('nginx')"  # 响应头检查很快
    - "status_code == 200"  # 状态码检查很快

# ❌ 性能较差的规则
slow-rule:
  dsl:
    - "regex(body, '.*very.*complex.*pattern.*')"  # 复杂正则很慢
```

### Q4: 如何处理中文内容？

**A**: AdventureX支持UTF-8编码，可以直接使用中文：

```yaml
chinese-site:
  dsl:
    - "contains(body, '用户登录')"
    - "contains(title, '管理系统')"
```

### Q5: 主动探测什么时候会触发？

**A**: 主动探测在以下情况触发：

1. 首次访问某个主机
2. 被动匹配没有结果
3. 存在包含`path`字段的规则

```yaml
# 这个规则会触发主动探测
api-check:
  path: "/api/info"
  dsl:
    - "contains(body, 'version')"
```

---

## 📝 总结

指纹规则编写是一门平衡艺术，需要在准确性、性能和可维护性之间找到平衡点。通过掌握DSL语法、理解匹配机制、遵循最佳实践，你可以编写出高效准确的指纹规则。

**记住核心原则**：

- 🎯 **准确性第一** - 确保规则匹配正确的技术
- ⚡ **性能考虑** - 优先使用高效的匹配方式
- 🔧 **易于维护** - 使用清晰的命名和适当的分类
- 📊 **充分测试** - 在实际环境中验证规则效果

希望这份指南能帮助你快速上手AdventureX指纹规则编写！ 
