## Vmail

使用 Cloudflare email worker 实现的临时电子邮件服务。

- 接收电子邮件（email worker）
- 显示电子邮件（remix）
- 邮件存储（sqlite）

> worker接收电子邮件 -> 保存到数据库 -> 客户端查询电子邮件

### 截图

demo：https://1010822.xyz

![](https://1010822.xyz/preview.png) 

## 自托管

### 准备工作

- [Cloudflare](https://dash.cloudflare.com/) 账户（email worker）
- 托管在 Cloudflare 上的域名
- [turso](https://turso.tech) sqlite（个人免费计划足够）

### 步骤

1.**注册一个 [turso](https://turso.tech) 账户，创建数据库，并创建一个`emails`表**

注册后，系统会提示您创建一个数据库。在这里我将其命名为 `vmail`，

![](https://img.inke.app/file/3773b481c78c9087140b1.png) 

然后，创建一个名为 `emails` 的表。

选择您的数据库，您会看到“编辑表”按钮，点击并进入:

![](https://img.inke.app/file/d49086f9b450edd5a2cef.png) 

> ⚠️ 注意：**左上角有一个加号按钮，我尝试点击它没有任何提示或效果，所以我使用了 turso 提供的 cli 来初始化表。**

Cli文档：https://docs.turso.tech/cli/introduction 

Linux (或 mac/windows) 终端执行：

```bash
# 安装（安装后记得重启终端生效）
curl -sSfL https://get.tur.so/install.sh | bash

# 登录账户
turso auth login

# 连接到您的Turso数据库
turso db shell <database-name>
```

将sql脚本复制到终端运行（packages/database/drizzle/0000_sturdy_arclight.sql）：

```sql
CREATE TABLE `emails` (
 `id` text PRIMARY KEY NOT NULL,
 `message_from` text NOT NULL,
 `message_to` text NOT NULL,
 `headers` text NOT NULL,
 `from` text NOT NULL,
 `sender` text,
 `reply_to` text,
 `delivered_to` text,
 `return_path` text,
 `to` text,
 `cc` text,
 `bcc` text,
 `subject` text,
 `message_id` text NOT NULL,
 `in_reply_to` text,
 `references` text,
 `date` text,
 `html` text,
 `text` text,
 `created_at` integer NOT NULL,
 `updated_at` integer NOT NULL
);
```

2.**部署 email worker**

```bash
git clone https://github.com/yesmore/vmail

cd vmail

# 安装依赖
pnpm install
```

在 `vmail/apps/email-worker/wrangler.toml` 文件中填写必要的环境变量。

- TURSO_DB_AUTH_TOKEN（第1步中的turso表信息，点击“Generate Token”）
- TURSO_DB_URL（例如 libsql://db-name.turso.io）

> 如果您不执行此步骤，可以在Cloudflare的 worker settings 中添加环境变量

然后运行命令：

```bash
cd apps/email-worker
pnpm run deploy
```

3.**配置电子邮件路由规则**

设置“Catch-all”动作为发送到emial worker。

![](https://img.inke.app/file/fa39163411cd35fad0a7f.png) 

4.**在 Vercel 或 fly.io 上部署 Remix 应用程序**

确保在部署期间准备并填写以下环境变量（`.env.example`）：

- COOKIES_SECRET（cookie的加密密钥，一个随机字符串即可）
- TURNSTILE_KEY（从Cloudflare获取，用于网站验证）
- TURNSTILE_SECRET
- TURSO_DB_RO_AUTH_TOKEN（从turso获取数据库凭据）
- TURSO_DB_URL

部署成功后在 cloudflare 添加域名解析到对应平台，就可以愉快的玩耍了。

以上，完成！

## License

GNU General Public License v3.0

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=yesmore/vmail&type=Date)](https://star-history.com/#yesmore/vmail&Date)

Inspired by smail.pw & email.ml