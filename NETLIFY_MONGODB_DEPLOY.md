# Cusdis 部署到 Netlify + MongoDB Atlas 指南

本指南将帮助您将 Cusdis 项目从 Vercel + PostgreSQL 迁移到 Netlify + MongoDB Atlas。

## 准备工作

### 1. MongoDB Atlas 设置

1. 访问 [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) 创建免费账户
2. 创建新的 Cluster (选择免费的 M0 Sandbox)
3. 设置数据库用户和密码
4. 配置网络访问（允许所有IP访问: 0.0.0.0/0）
5. 获取连接字符串，格式如下：
   ```
   mongodb+srv://username:password@cluster.mongodb.net/cusdis?retryWrites=true&w=majority
   ```

### 2. 创建 Netlify 账户

1. 访问 [Netlify](https://netlify.com) 创建账户
2. 连接您的 Git 仓库

## 部署步骤

### 1. 更新项目配置

项目已经配置为使用 MongoDB，相关文件已更新：
- ✅ `prisma/mongodb/schema.prisma` - MongoDB数据库schema
- ✅ `package.json` - 构建脚本已更新为使用MongoDB
- ✅ `netlify.toml` - Netlify部署配置

### 2. 配置环境变量

在 Netlify 项目设置中添加以下环境变量：

#### 必需的环境变量：

```bash
# 数据库连接 (必需)
DATABASE_URL=mongodb+srv://username:password@cluster.mongodb.net/cusdis?retryWrites=true&w=majority

# Next Auth 配置 (必需)
NEXTAUTH_URL=https://your-netlify-app.netlify.app
NEXTAUTH_SECRET=your-nextauth-secret-key
JWT_SECRET=your-jwt-secret-key

# 应用配置 (必需)
HOST=https://your-netlify-app.netlify.app
IS_HOSTED=true
```

#### 认证提供商 (至少选择一种)：

```bash
# 方式1: 简单用户名/密码登录
USERNAME=admin
PASSWORD=your-secure-password

# 方式2: GitHub OAuth
GITHUB_ID=your-github-client-id
GITHUB_SECRET=your-github-client-secret

# 方式3: GitLab OAuth  
GITLAB_ID=your-gitlab-client-id
GITLAB_SECRET=your-gitlab-client-secret

# 方式4: Google OAuth
GOOGLE_ID=your-google-client-id
GOOGLE_SECRET=your-google-client-secret
```

#### 邮件配置 (选择一种)：

```bash
# 方式1: SMTP (如Gmail)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_SECURE=true
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password
SMTP_SENDER=Your App Name<your-email@gmail.com>

# 方式2: SendGrid
SENDGRID_API_KEY=your-sendgrid-api-key
```

#### 可选的环境变量：

```bash
# 错误跟踪
SENTRY_DSN=your-sentry-dsn

# 分析
UMAMI_ID=your-umami-id
UMAMI_SRC=your-umami-script-src

# 付费功能 (LemonSqueezy)
CHECKOUT_URL=your-checkout-url
LEMON_SECRET=your-lemon-webhook-secret
LEMON_API_KEY=your-lemon-api-key
```

### 3. 在 Netlify 中部署

1. 将代码推送到您的 Git 仓库
2. 在 Netlify 中连接仓库
3. 构建设置会自动从 `netlify.toml` 读取
4. 在环境变量设置页面添加上述所有必需的环境变量
5. 触发部署

### 4. 安装 Netlify 插件

确保在您的 Netlify 项目中安装了以下插件：
- `@netlify/plugin-nextjs`

### 5. 数据库初始化

部署成功后，数据库表会自动创建（通过 `npm run db:push`）。

## 重要说明

### 与原版本的差异

1. **数据库类型**: 从 PostgreSQL 改为 MongoDB
2. **ID字段**: 使用 MongoDB ObjectId 替代 UUID/自增ID
3. **部署平台**: 从 Vercel 改为 Netlify
4. **构建流程**: 使用 `db:push` 替代 `db:migrate`

### 数据迁移

如果您有现有的 PostgreSQL 数据需要迁移：

1. 从原数据库导出数据
2. 使用项目内置的数据导入功能（支持 Disqus 格式）
3. 或者编写自定义迁移脚本

### 故障排除

#### 常见问题：

1. **构建失败**: 检查所有必需的环境变量是否正确设置
2. **数据库连接失败**: 验证 `DATABASE_URL` 格式和网络访问设置
3. **认证问题**: 确保至少配置了一种认证方式
4. **邮件发送失败**: 检查 SMTP 或 SendGrid 配置

#### 调试技巧：

1. 查看 Netlify 的构建日志
2. 检查函数日志以获取运行时错误
3. 验证环境变量是否正确传递

## 性能优化建议

1. **MongoDB索引**: 部署后考虑为常用查询添加索引
2. **CDN**: Netlify 自动提供 CDN 加速
3. **缓存**: 考虑使用 Redis 作为缓存层（可选）

## 支持

如果遇到问题，请检查：
1. Netlify 构建日志
2. MongoDB Atlas 连接日志
3. 浏览器开发者工具的网络和控制台 