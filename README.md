# vultr coolify：用 Marketplace 镜像免手动安装，从选配置到部署第一个应用的全流程

在 Vultr 上装 Coolify，不用自己敲安装命令。Vultr 官方提供 Coolify Marketplace 应用——一个预配置好的服务器镜像，选它开机，访问服务器的 8000 端口就能进入 Coolify 面板。Coolify 本身是开源软件，不收授权费，你实际付费的只有 Vultr 的服务器。

这篇解决的是搜 vultr coolify 的人最常卡住的几件事：配置买多大、镜像怎么选、开机后第一步做什么、没有域名能不能先测。按实际操作顺序走一遍。

> **选购速览**
> - 部署路径：Products → Compute → Deploy Server，在 Marketplace Apps 里选 Coolify，开机即用，无需手动安装
> - 配置参考：单个容器化应用按 2 GB 内存 / 1 核起步；带数据库的多服务项目按 4 GB 内存 / 2 核考虑
> - 首次访问：http://服务器IP:8000，注册管理员账号后进入面板
> - 👉 [前往 Vultr 部署 Coolify Marketplace 应用](https://www.vultr.com/marketplace/apps/coolify/?ref=7268283)

## Vultr 的 Coolify Marketplace 应用是什么

它是 Vultr 官方整理好的 Coolify 服务器镜像。选它当系统镜像开一台服务器，Coolify 和它依赖的运行环境在开机时就已经就绪，省掉的是装 Docker、配面板这些步骤。按官方文档的说法，几分钟就能得到一台可用的 Coolify 实例。

Coolify 本身解决的是另一层问题：把应用部署和运维收进一个面板。连上 Git 仓库后，代码推送就能触发构建和部署；反向代理、TLS 证书、数据库这类常驻组件由它统一管理。

具体到功能，官方列出来的包括：

- 支持多种语言和框架：静态网站、API、后端服务、数据库都能部署
- 从 GitHub、GitLab、Bitbucket 拉代码，推送即部署，也提供 API 接 CI/CD
- Let's Encrypt 免费证书自动配置和管理，数据库备份可存到任意 S3 兼容存储
- 浏览器里的实时终端，不开 SSH 客户端也能操作服务器
- 监控服务器健康、磁盘用量和部署状态，异常时通过 Discord、Telegram 或邮件提醒
- 一键部署预配置服务（比如 NextCloud），也支持自己写 Docker 配置

顺带说一句，Coolify 的部署目标是任何能 SSH 连上的服务器，它不把你锁在 Vultr 上，将来换机器、加机器，面板和部署流程是通用的。所以选 Vultr 还是别家，决定的是服务器本身的价格、机房和稳定性，跟部署方式关系不大。

## 配置怎么选：官方文档给过两个“最低要求”

这里有个容易被忽略的细节：Vultr 官方不同教程里写的 Coolify 最低配置并不一样。

部署 v0 或 Bolt.new 生成的项目，文档写的是**至少 2 GB 内存、1 个 CPU 核心**；而 Lovable 项目的部署指南写的是**至少 4 GB 内存、2 个 CPU 核心**。差别出在项目形态上——AI 生成的全栈应用经常带独立数据库容器，多个容器挤在一台机器上，内存消耗明显更高。

| 使用场景 | 参考最低配置 | 说明 | 购买入口 |
| --- | --- | --- | --- |
| 单个容器化应用（静态站、单个 API） | 2 GB 内存 / 1 vCPU | v0、Bolt.new 项目部署指南给出的下限 | [ 部署 Coolify](https://www.vultr.com/marketplace/apps/coolify/?ref=7268283) |
| 多服务项目（应用 + 独立数据库容器） | 4 GB 内存 / 2 vCPU | Lovable 项目部署指南给出的下限 | [ 部署 Coolify](https://www.vultr.com/marketplace/apps/coolify/?ref=7268283) |
| 团队项目、多环境并存 | 官方未给固定数字，按应用数量往上加 | 同一项目可建 production、staging 等多个环境 | [ 部署 Coolify](https://www.vultr.com/marketplace/apps/coolify/?ref=7268283) |

👉 [查看 Vultr 当前的服务器套餐与价格](https://bit.ly/VulTR)

机房选离目标用户近的，Vultr 一共有 33 个位置可选。服务器类型上，官方部署教程用的是 Dedicated CPU。价格随套餐和机房浮动，下单页会显示实际数字。

## 三种部署方式：控制台、API 和 CLI

绝大多数人在控制台点几下就完成了，但 Vultr 同时保留了命令行通道，批量开机器或者用基础设施即代码管理服务器时用得上。

| 部署方式 | 关键操作 | 入口 |
| --- | --- | --- |
| 控制台手动部署 | Products → Compute → Deploy Server，选服务器类型、机房、套餐，在 Marketplace Apps 里搜 Coolify，确认配置后点 Deploy | [ 用 Coolify 镜像开一台服务器](https://www.vultr.com/marketplace/apps/coolify/?ref=7268283) |
| Vultr API / Terraform | 实例参数里把 image_id 设为 coolify | [ 部署 Coolify](https://www.vultr.com/marketplace/apps/coolify/?ref=7268283) |
| Vultr CLI | vultr-cli instance create 命令加 --image "coolify" | [ 部署 Coolify](https://www.vultr.com/marketplace/apps/coolify/?ref=7268283) |

部署完成后，到 Compute 列表点进实例，记下 IP 地址，后面访问面板和绑定域名都要用它。

## 开机后第一件事：进 8000 端口注册管理员

Coolify 的面板跑在服务器的 8000 端口。打开实例的管理页，往下滚到 App Instructions 区域，能看到完整的访问地址，格式是 http://服务器IP:8000。浏览器打开，首次访问是注册页，填名称、邮箱和密码创建管理员账号。这个账号控制之后所有的部署、团队成员和服务器设置，密码别随手设。

注册完会问要不要走引导。熟悉这类面板的直接 Skip Onboarding 进主界面；第一次用建议跟一遍 Get Started 向导，它会带你建第一个项目、选部署目标。

向导里有个容易让人迟疑的选项：部署目标选 Localhost。它指的是这台 Vultr 实例上的 Docker，也就是你的应用默认运行的地方，不需要也没有必要在本地电脑装任何东西。

## 部署第一个应用：连 Git、选构建包、用临时域名

应用部署的完整链路：

1. 在面板里进 Sources，点 Add Source 选 GitHub，跳转完成 OAuth 授权，之后 Coolify 就能读取你的仓库
2. 进 Projects，Add Project 建项目；项目会自动带上一个默认的 production 环境
3. 项目里点 Add Resource，资源类型选 Application，来源选 GitHub，下拉框里挑仓库，分支一般填 main
4. Coolify 会扫描仓库识别容器配置；单容器项目走 Dockerfile，多容器项目把构建包选成 Docker Compose
5. 保存后开始构建部署，进度和状态在面板里可以直接看

有两件事能帮你省时间、省成本。

一是不用先买域名。部署完成后 Coolify 会分配一个临时域名，格式是 RANDOM_ID.服务器IP.sslip.io，浏览器直接打开就能测试。等应用跑通了，再给正式域名加一条 A 记录指向服务器 IP 就完成绑定，证书由 Let's Encrypt 自动处理。

二是敏感信息放环境变量里，别写进代码。应用详情页进 Environment Variables，点 +Add 添加。Coolify 会加密存储这些值，容器启动时注入。常见的几类：

- DATABASE_URL：数据库连接串，比如 postgresql://user:pass@db:5432/dbname
- API_KEY：外部服务的密钥
- NODE_ENV：运行环境，一般填 production
- NEXT_PUBLIC_API_URL：前端调用的 API 地址

## 日常真正会用到的东西

应用跑起来之后，高频用到的是这几块：

- 自动 SSL：Let's Encrypt 免费证书自动配置，不用手动申请
- 数据库备份：定时备份存到任意 S3 兼容存储
- 监控和告警：服务器健康、磁盘用量、部署状态，出问题通过 Discord、Telegram 或邮件推给你
- 一键服务：预配置服务（比如 NextCloud）一键起，自定义 Docker 服务也支持
- 浏览器终端：临时进服务器排查问题时，不用另开 SSH 工具

## 几个常见问题

**Coolify 本身收费吗？**
不收。Coolify 是开源项目，免费使用，费用只产生在 Vultr 服务器上。

**最低买多大配置？**
官方文档对 v0、Bolt.new 这类项目写的是 2 GB 内存、1 核；带独立数据库容器的多服务项目，Lovable 指南给的下限是 4 GB 内存、2 核。不确定自己属于哪种时，按 4 GB 预算更稳。

**还没买域名，能先测吗？**
能。部署后 Coolify 分配 sslip.io 格式的临时域名，测试阶段够用，正式域名以后加 A 记录再绑。

**应用会被锁死在 Vultr 上吗？**
不会。Coolify 支持向任何能 SSH 连接的服务器部署，包括自有机器和其他云。迁移时的工作量在数据和应用配置本身，面板流程是通用的。

## 按你的情况挑方案

- 只想跑一两个小应用练手：2 GB 内存档够用，重点是把 Marketplace 镜像选成 Coolify
- 跑 AI 生成的全栈项目、带数据库：直接按 4 GB 内存 / 2 核配，省得回头停机升配
- 团队多人用、要分环境：项目和环境结构本身不额外花钱，内存按应用数量往上加

下单前确认两件事：机房离你的用户近不近；内存别卡着最低线买，多容器应用的消耗很容易超出预期。想清楚这两点，就可以 [👉 去 Vultr 挑一台装 Coolify 的服务器](https://bit.ly/VulTR) 开机了。
