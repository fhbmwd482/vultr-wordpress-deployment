# vultr one click wordpress：分步设置、免费 SSL、管理后台登录详情，以及大多数指南都忽略的升级陷阱

Vultr 的 One-Click WordPress 应用能在一分钟内上线一台预配置好的 WordPress 服务器。选择服务器位置，选定套餐规格，点击 Deploy Now，平台就会启动一台已经安装好 Nginx、MariaDB、PHP 和 WordPress 的实例——此外还内置了 Certbot、Wordfence 和基于 Web 的服务器面板，许多人往往在后期才发现这些工具。

但这正是编写本指南的原因。一键部署涵盖了服务器搭建，而从部署到网站正式上线之间的每一步——包括安全弹窗、浏览器证书警告、设置向导，以及 SSL 证书中一个与 DNS 相关的陷阱——都需要你来完成。Vultr 还有一项让用户在一年后才感到意外的升级政策：已部署的 One-Click 服务器永远不会自动打补丁。在投入之前，这两点都值得了解。

> **快速总结：** 对于博客、小型企业网站或 modest 店铺，请在 Vultr 较小的 Cloud Compute 套餐上部署 One-Click WordPress 应用。该应用是免费的——你只需支付服务器费用——并且可以通过向导在几分钟内完成设置。👉 [在 Vultr 上部署 One-Click WordPress 应用](https://www.vultr.com/marketplace/apps/wordpress/?ref=7268283) 开始使用；点击 Deploy 后大约一分钟，你就会进入设置向导。

## 部署是如何工作的

部署过程设计得很简短。选择一个靠近你受众的服务器位置，选择一个套餐规格，然后点击 Deploy Now。根据 Vultr 自己的文档，配置大约需要一分钟，之后服务器就会启动，等待你在浏览器中完成 WordPress 设置。

你需要的每个密码都在 Vultr 控制台的服务器信息页面上。该页面列出了用于 SSH 访问的 root 登录方式，并在 Application Information 下提供了 WordPress 管理区域的凭据。该应用的 Marketplace 列表还显示了一个预创建的管理员账户，用户名为 `wpauser`。在完成本指南的其余部分时，请保持该信息页面打开——几乎每个步骤都会用到它。

## 在 Vultr 上运行 WordPress 的三种方式

One-Click WordPress 应用是默认答案，但不是唯一途径。Vultr 为店铺提供了匹配的 WooCommerce 镜像，如果你更愿意自己安装技术栈，也可以选择纯粹的 Cloud Compute 服务器。定价在每个案例中都是一样的：应用镜像本身不收费，你只需为附加的计算套餐支付费率，最小的规格起价每月只需几美元。

| 方式 | 你的初始配置 | 适用场景 | 部署 |
| --- | --- | --- | --- |
| One-Click WordPress | Nginx + MariaDB + PHP + WordPress，预装 Certbot、SMTP Mailer、Wordfence、Cockpit、phpMyAdmin | 博客、企业网站、大多数店铺 | [ 部署 WordPress](https://www.vultr.com/marketplace/apps/wordpress/?ref=7268283) |
| One-Click WooCommerce | 相同的技术栈，且从一开始就包含 WooCommerce 功能 | 希望预配置商店层的店主 | [ 部署 WooCommerce](https://www.vultr.com/apps/woocommerce?ref=7268283) |
| 纯 Cloud Compute | 空白服务器；你需自行安装和维护所有内容 | 自定义技术栈，需要完全控制权的开发者 | [ 查看 Vultr 的套餐](https://bit.ly/VulTR) |

本指南的其余部分遵循 One-Click WordPress 路径；根据 Vultr 的文档，WooCommerce 用户遵循相同的步骤。

## 首次登录：进入仪表板前的两道关卡

预判会有两次登录，且不要将它们混淆。第一道关卡是保护 `/wp-admin/` 的浏览器弹窗——Vultr 在 WordPress 设置时增加的这一层保护需要你输入服务器信息页面上的安全用户凭据。第二道关卡是 WordPress 自身的登录界面，在这里使用预创建的 `wpauser` 账户（或在向导中创建的管理员账户）。

如果你尚未安装 SSL 证书，浏览器会抛出警告，因为服务器附带的是自签名证书。根据 Vultr 的文档，此时继续操作是安全的。通过纯 HTTP 访问 wp-admin 也可以，但这会以未加密方式发送你的密码，因此请将其视为最后手段。安装完成后，可以移除该弹窗——Vultr 的 One-Click WordPress 文档中的“Remove wp-admin Security”部分对此进行了说明。

## 设置向导

大约四屏。选择语言并点击 Continue。输入网站标题、管理员用户名和电子邮件地址；WordPress 会建议一个强密码，在点击 Install WordPress 之前值得将其记下。等待安装程序完成，登录，然后你就会进入仪表板。

给迁移现有网站的人一个提示：在导入旧内容时，你在这里输入的任何内容都会被覆盖，所以不必纠结于标题。不过密码依然很重要，因为它是你第一次登录时的凭据。

## 域名、DNS 和免费 SSL 证书

顺序很重要：先域名，后证书。注册一个域名，创建一条 DNS 记录将其指向服务器的 IP 地址，并确认记录已传播——DNS 通常很快，但有时可能需要长达 48 小时。Certbot 只有在域名实际解析到服务器后才能签发证书，因此过早运行命令是此步骤最常见的失败原因。

域名解析完成后，以 root 身份通过 SSH 连接并运行一个命令：


certbot --nginx --redirect -d www.example.com -d example.com -m admin@example.com --agree-tos --no-eff-email


替换为你自己的域名和电子邮件。同时列出裸域名和 www 版本会创建一个涵盖两者的单一多域名证书，`--redirect` 标志会让 Nginx 强制使用 HTTPS。Certbot 在该应用上已预装，完成后应报告成功。

有两个边缘情况值得了解。如果你现在跳过 SSL 并在几个月后添加它，WordPress 在此期间会存储未加密的链接——使用流行的 HTTPS 插件之一可以强制所有内容回到 HTTPS 并清理它。如果你更喜欢商业证书，该应用要求将公有证书放在 `/etc/nginx/ssl/server.crt`，私钥放在 `/etc/nginx/ssl/server.key`，然后重启。

## 将 WordPress 指向你的域名

证书放置好后，告诉 WordPress 它自己的地址。在仪表板中，转到 Settings > General，将 WordPress Address (URL) 和 Site Address (URL) 均设置为你的域名，然后保存。

在创建内容之前执行此操作是有原因的：WordPress 将完整的 URL 存储在数据库中。如果网站仍在服务器 IP 上运行时你创建了页面，稍后切换时，某些链接将继续指向旧地址。Vultr 的文档建议如果发生这种情况，使用备份和数据库搜索替换插件——如果早点设置好 URL，这本来是可以避免的。

## 服务器上预装的其他所有内容

该镜像不仅包含 WordPress，而且其中一些工具在你知道它们存在之前就已经在运行。以下是完整清单以及每个组件的所在位置：

| 工具 | 用途 | 所在位置及使用方法 |
| --- | --- | --- |
| Certbot | 签发和续订免费的 Let's Encrypt 证书 | 预装；通过 SSH 运行 `certbot --nginx ...` |
| SMTP Mailer | 通过你自己的 SMTP 服务器发送 WordPress 邮件，解决了常见的邮件投递难题 | Plugins > Installed Plugins；激活后设置链接会出现在插件名称下 |
| Wordfence | 端点防火墙和恶意软件扫描程序；Vultr 建议激活它 | Plugins > Installed Plugins |
| Cockpit | 基于 Web 的服务器管理面板 | 通过 HTTPS 访问端口 9080 (your-domain:9080)；凭据在 Server Information 页面；在 Vultr 防火墙中允许端口 9080 |
| phpMyAdmin | 数据库管理 | 位于服务器 IP 的 `/mysqladmin` 路径下——用户名为 `pmauser`，浏览器弹窗和 phpMyAdmin 本身有不同的密码 |
| MariaDB | WordPress 运行所在的数据库 | root 密码存储在服务器上的 `/root/.my.cnf` 中；通过 `journalctl -eu mariadb.service` 查看日志 |
| Linux Malware Detection | 恶意软件扫描；默认启用 | 使用 `systemctl disable --now maldet.service` 禁用 |

并非每个工具都有用。如果你不使用 Cockpit，可以通过以 root 身份连接并运行 `systemctl disable --now cockpit.socket` 将其完全关闭——这样可以减少一个监听在公共端口上的服务。Cockpit 也可以共享你的 Let's Encrypt 证书：一旦签发了主证书，位于 `/opt/vultr/cockpit-certificate.sh` 的脚本就会将其应用到面板上。

## 大多数指南都忽略的升级陷阱

这是改变长期账本的部分。Vultr 每年都会多次用软件更新和安全补丁刷新其 One-Click 应用镜像，但这些刷新仅影响新的部署。你去年部署的服务器将保持原样。在已部署的 One-Click 实例上修补单个 PHP 或应用组件也不受支持，因此获得最新、安全的 WordPress 的官方途径是部署新服务器并进行迁移。

这没有听起来那么痛苦，而且它也可以作为将网站从其他主机迁移到 Vultr 的步骤：

1. 在进行任何操作之前，先对旧服务器进行快照。如果是来自其他主机，请使用镜像备份或诸如 UpdraftPlus 或 Jetpack 之类的插件。
2. 在旧网站上安装 All-in-One WP Migration 插件，运行 Export to File，然后下载导出文件。免费版本涵盖了大多数网站。
3. 关闭旧服务器——暂时不要销毁它。
4. 👉 [部署全新的 One-Click WordPress 服务器](https://www.vultr.com/marketplace/apps/wordpress/?ref=7268283)，使用临时值运行向导，然后登录。
5. 更新 DNS 以指向新服务器。如果保留旧地址更方便，使用保留 IP 可以继续使用该地址。
6. 在新服务器上安装 All-in-One WP Migration，导入文件，确认，并在提示时保存固定链接结构。
7. 使用 Certbot 安装 SSL 证书，测试网站，然后销毁旧服务器。

为此预留一个维护窗口——在导出和 DNS 切换之间，网站将处于离线状态。这就是为了无需手动维护服务器技术栈而做出的折衷；是否值得取决于你能容忍多少停机时间以及你希望多久进行一次此操作。

## One-Click 应用适合谁

对于任何想要标准 WordPress 网站而无需手动组装技术栈的人来说，One-Click 应用是 Vultr 上的合理默认选择：部署大约需要一分钟，向导处理 WordPress 部分，Certbot 加上预装插件集覆盖了通常的第一天琐事。

有两种情况需要三思。如果你希望服务器端永远自动为你修补，此应用不会做到——迁移到升级是上述的模式，诚实地问自己是否每隔几年真的会做一次是值得的。如果你的项目实际上是商店优先，网站其次，那么 One-Click WooCommerce 应用会为你提供开箱即用的商店功能：👉 [部署 One-Click WooCommerce 应用](https://www.vultr.com/apps/woocommerce?ref=7268283) 并遵循相同的设置步骤。

## 常见问题

### 整个设置需要多长时间？

部署大约需要一分钟。向导增加了几分钟，SSL 证书完全取决于 DNS：一旦你的域名传播完成——通常很快，偶尔长达 48 小时——Certbot 会在一分钟内完成。

### 我可以在没有域名的情况下使用它吗？

可以。该网站通过服务器的 IP 地址工作，在设置过程中可以安全地绕过自签名证书警告。在准备好后添加域名和 SSL；只需在创建太多内容之前，确保在 Settings > General 中设置好 URL。

### 所有的密码都在哪里？

在 Vultr 控制台的服务器信息页面上：用于 SSH 的 root 登录、wp-admin 弹窗的安全用户凭据、预创建的 `wpauser` WordPress 管理员、Cockpit 凭据，以及 phpMyAdmin 的用户名和密码。MariaDB root 密码是独立的——它存储在服务器上的 `/root/.my.cnf` 中。

### Vultr 会帮我保持 WordPress 更新吗？

不会。Vultr 会刷新用于新部署的应用镜像，但现有服务器什么也得不到，且在已部署的实例上修补单个组件是不受支持的。保持最新意味着部署一个新实例并按上述步骤进行迁移。

### 如果我不需要某些东西，可以关闭它们吗？

可以。Cockpit 通过 `systemctl disable --now cockpit.socket` 关闭，Linux Malware Detection 通过 `systemctl disable --now maldet.service` 关闭。两者都是可逆的。

## 在你点击 Deploy 之前

有三件事需要解决：一个 DNS 记录已指向你的服务器（或耐心等待传播）的域名、一个书签保存好的 Server Information 页面，以及你对升级模式的决定——如果你从不打算进行迁移，最好现在就知道，而不是在两年后。

应用本身是免费的，所以剩下的唯一决定就是套餐规格，从小规格开始是常规做法。👉 [查看 Vultr 当前的 Cloud Compute 套餐并部署 One-Click WordPress 应用](https://bit.ly/VulTR)，随时准备开始。
