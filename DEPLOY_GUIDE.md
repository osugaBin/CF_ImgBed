# Cloudflare ImgBed 部署指南

## 前置准备

1. Fork [MarSeventh/CloudFlare-ImgBed](https://github.com/MarSeventh/CloudFlare-ImgBed)
2. 准备 Cloudflare 账户

---

## 一、创建 Cloudflare Pages 项目

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. **Workers & Pages** → **创建应用程序** → **导入现有 Git 仓库**
3. 连接你的 Fork 仓库
4. 配置：

| 配置项 | 值 |
|--------|-----|
| 项目名称 | `cf-imgbed`（自定义） |
| 生产分支 | `main` |
| 构建命令 | `echo "skip"` |
| 构建输出目录 | `/` |

5. 点击 **保存并部署**

---

## 二、创建 R2 存储桶

⚠️ **必须用 Wrangler CLI 创建，Dashboard 创建会强制要求 jurisdiction**

```bash
npx wrangler r2 bucket create img-r2-new
```

⚠️ **注意**：bucket 名称不能使用下划线，必须用连字符 `-`

---

## 三、创建 KV Namespace

```bash
npx wrangler kv:namespace create "img_url"
```

记录返回的 `id`

---

## 四、添加 wrangler.toml

在项目根目录创建 `wrangler.toml`：

```toml
name = "cf-imgbed"
compatibility_date = "2024-04-23"
pages_build_output_dir = "."

[[r2_buckets]]
binding = "img_r2"
bucket_name = "img-r2-new"

[[kv_namespaces]]
binding = "img_url"
id = "你的KV_ID"
```

推送到 GitHub：

```bash
git add -f wrangler.toml
git commit -m "Add wrangler.toml"
git push
```

---

## 五、Dashboard 绑定配置

### R2 Bucket 绑定

1. **Settings** → **Functions** → **R2 Bucket Bindings**
2. 添加绑定：
   - 变量名称：`img_r2`
   - R2 存储桶：`img-r2-new`
   - **Jurisdiction：留空不选**

### KV Namespace 绑定

1. **Settings** → **Functions** → **KV Namespace Bindings**
2. 添加绑定：
   - 变量名称：`img_url`
   - KV 命名空间：选择 `img_url`

---

## 六、环境变量配置

| 变量 | 值 | 说明 |
|------|-----|------|
| `TG_BOT_TOKEN` | 你的 Telegram Bot Token | 必须 |
| `TG_CHAT_ID` | 你的 Telegram Chat ID | 必须 |

---

## 七、用 Wrangler CLI 部署

```bash
npx wrangler pages deploy . --project-name=cf-imgbed
```

---

## 八、自定义域名（可选）

**Settings** → **Custom domains** → **设置自定义域**

---

## R2 Bucket 位置说明

| location | 含义 |
|----------|------|
| `WNAM` | 美国西部 |
| `ENAM` | 美国东部 |
| `WEUR` | 西欧 |
| `EEUR` | 东欧 |
| `APAC` | 亚太 |

---

## 常见问题

### Q: 报错 `invalid jurisdiction`
**原因**：绑定的 R2 bucket 创建时选择了 jurisdiction

**解决**：创建新的 bucket，不选择 jurisdiction：
```bash
npx wrangler r2 bucket create img-r2-new
```

### Q: 部署后上传失败
**解决**：
1. 确认 R2 和 KV 绑定正确
2. 用 Wrangler CLI 重新部署：
   ```bash
   npx wrangler pages deploy . --project-name=cf-imgbed
   ```

---

## 后续更新

| 更新内容 | 方式 |
|---------|------|
| 静态文件（CSS/JS/图片） | `git push` 自动部署 |
| Functions | `npx wrangler pages deploy . --project-name=cf-imgbed` |

---

## 参考

- 原项目：[MarSeventh/CloudFlare-ImgBed](https://github.com/MarSeventh/CloudFlare-ImgBed)
- 官方文档：[cfbed.sanyue.de](https://cfbed.sanyue.de)
