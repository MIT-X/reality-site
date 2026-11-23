# REALITY 本机 HTTPS 网站一键部署

[![Linux compatibility](https://github.com/tinkerlab-tech/reality-site/actions/workflows/compatibility.yml/badge.svg)](https://github.com/tinkerlab-tech/reality-site/actions/workflows/compatibility.yml)

为 VLESS＋REALITY 搭建本机 HTTPS 目标网站。中文交互、网页内置，证书由 Caddy 自动申请和续期。节点协议和入站通过 3X-UI 配置。

## 下载并运行

在 VPS 的 **root 终端**执行，需要已安装 `curl`：

```bash
curl -fL https://raw.githubusercontent.com/MIT-X/reality-site/main/reality-site.sh -o /root/reality-site.sh && bash /root/reality-site.sh
```

更新或修复本脚本部署的同一站点，也执行上面这条命令。脚本自动完成必要检查，通过后直接询问域名、本机网站端口和可选邮箱；核对计划后输入 `DEPLOY`。

## 准备与演示顺序

1. 准备 VPS，将域名的 A/AAAA 解析到这台 VPS，关闭 CDN 代理。
2. 安装 3X-UI 并确认能登录；已有面板可以继续使用。
3. 运行本脚本，等待本机 HTTPS 验证成功。
4. 在 3X-UI 配置 VLESS＋REALITY＋Vision，使用脚本输出的 SNI、Target/Dest 和 Xver。
5. 验证公网 `https://你的域名/`，再验证真实客户端代理连接。

只有本机 HTTPS 验证成功时，公网 443 的 REALITY 回落和代理出站仍需分别验证。

## 自定义网页

部署成功后，通过SSH工具的SFTP文件管理器，将自制网页上传到VPS的 **`/var/www/reality-site/`**。首页文件名为 **`index.html`**，可以直接拖拽替换。

可先下载[可编辑网页模板](site/index.html)，在电脑上修改标题、简介和文章内容、预览效果，再上传；也可以换成自己的个人博客、摄影作品或项目介绍等静态网页。

```text
/var/www/reality-site/
├── index.html
├── about.html
├── css/style.css
└── images/photo.jpg
```

- 先部署再上传；把网站文件夹里的内容放进上述目录，让`index.html`直接位于目录下。
- 支持HTML、CSS、JavaScript和图片，保持资源的相对路径。前端项目上传构建后的静态文件。
- 保存上传后刷新浏览器即可，无须重跑脚本、重新签证书或重启Caddy；有缓存时强制刷新。
- 更新部署脚本会保留已有网页；只有首页缺失时才生成默认页。
- PHP、WordPress和后端接口需要另行部署；此处是静态网站目录。出现403时检查文件可读、目录可进入，常用权限分别为644、755。

`site/index.html`是当前内置默认页的可编辑版本，自定义完成后上传到网页目录即可。默认部署仍只需下载脚本一个文件。

## 端口与证书

| 端口 | 用途 |
| --- | --- |
| 公网 TCP 80 | HTTP-01 证书验证时临时使用，完成后释放；申请和续期时均需可达、可用 |
| 公网 TCP 443 | 留给 Xray；在面板配置入站 |
| 本机 127.0.0.1:8443 | HTTPS 网站；首次占用时推荐其他本机端口，最终以输出的 Target 为准 |
| 本机 2019 | Caddy 管理接口 |

网站只提供 HTTPS，不提供常驻 HTTP 网页或自动跳转。浏览器明确输入 `https://`；需要先配置好公网 443 的 Xray 入站。

只使用 Caddy＋HTTP-01，无 DNS API、acme.sh 或额外续期定时任务。脚本保留 SSH、防火墙、3X-UI 和 Xray 原配置。

## 已安装 Caddy 或端口被占用

- **由本脚本部署、配置未被人工修改**：识别部署记录后复用安装、端口、网页和证书缓存，备份配置后更新并重载。
- **其他来源安装的 Caddy**：安装前说明原因并退出，保留现有网站，需要人工合并配置。
- **本脚本的 Caddyfile 被人工修改**：停止，先人工合并修改。
- **80 被其他服务占用**：显示占用进程并退出，人工协调后再运行。
- **旧部署使用其他证书验证方式**：保留原配置和续期任务，先人工迁移。

## 系统适配

按 VPS 实际安装的系统识别，不限制搬瓦工、DMIT、VMISS 等商家。

| 系统 | 脚本接受的版本 |
| --- | --- |
| Ubuntu | 20.04、22.04、24.04、26.04 |
| Debian | 11、12、13 |
| CentOS Stream | 9、10 |
| AlmaLinux、Rocky Linux、RHEL、Oracle Linux | 8、9、10，包括对应小版本 |
| Fedora | 43、44、45 |

架构为 x86_64 或 aarch64，要求 systemd、root，以及可用的系统软件源。自动区分 DNF4/DNF5 的 COPR 插件，并为 RPM 系统指定对应的 EL/Fedora 与 CPU 架构软件源。依赖安装或软件源启用失败即停止，不跨版本混用软件包。

CentOS Linux 7/8、CentOS Stream 8、Debian 10、Alpine/OpenRC、Arch 和 openSUSE 尚未适配；未知版本直接说明原因并退出。精简镜像缺少必要基础工具时会列出对应工具包，不会自动重装系统或切换到归档源。

这些是已实现安装分支的兼容目标，**不代表每个系统都已完成实机安装**。选型参考[搬瓦工镜像公告](https://bandwagonhost.com/news)、[DMIT公开系统列表](https://www.dmit.io/pages/cloud-instance?language=english)；VMISS的完整镜像版本列表未能从公开页面确认。安装方式依据[Caddy官方说明](https://caddyserver.com/docs/install)及[Caddy COPR构建目录](https://download.copr.fedorainfracloud.org/results/@caddy/caddy/)，商家后台可选版本仍以实际面板为准。

## 排查

查看 Caddy 日志：

```bash
journalctl -u caddy -n 60 --no-pager
```

检查是否有常驻 80 监听：

```bash
ss -lntp 'sport = :80'
```

没有证书验证任务或其他 HTTP 服务时，80 应只显示表头。证书问题先检查域名全部 A/AAAA、公网 80 可达性和 CAA；Caddy 会自动重试，无须反复安装或强制重签。

## 验证范围

### GitHub Actions 自动兼容测试

[查看每个系统的真实测试日志](https://github.com/tinkerlab-tech/reality-site/actions/workflows/compatibility.yml)。修改脚本或测试后自动运行，也可在Actions中选择Linux compatibility → Run workflow手动运行。

**2026年11月18日：[完整一轮45/45通过](https://github.com/tinkerlab-tech/reality-site/actions/runs/44261853434)，对应提交`6e521d1`。** 真实安装测试发现并修复了精简RPM镜像中`curl-minimal`与完整`curl`的冲突：已有curl时直接复用，缺少时才安装。后续以新提交的测试结果为准。

该轮使用本地CA，未覆盖真实HTTP-01监听。后续首次签发实测暴露：站点的`bind 127.0.0.1`被Caddy继承为ACME挑战监听地址，公网80因而拒绝连接。已通过全局ACME签发器、CA和默认绑定设置分离验证监听与网站监听；网站仍只绑定本机，挑战使用公网通配地址。新增本地ACME测试实际访问非回环IPv4及可用的IPv6挑战地址，旧版可复现失败、修复版本地通过；该测试已接入每个安装任务。验证超时也不再追加误导性的“安装失败行号”。

**补齐HTTP-01检查后：[45/45组合均通过](https://github.com/tinkerlab-tech/reality-site/actions/runs/44278120303)，对应提交`8cf9d7b`。** Ubuntu 24.04的一次任务停在系统镜像下载阶段，单项重跑后通过；其余结果保留。挑战测试使用临时高端口及本地模拟ACME服务，不申请公网证书；用户随后在Debian 13实际完成公网CA签发及本机HTTPS验证。

- **42个安装任务**：21种系统版本 × x86_64/ARM64，使用对应发行版容器和原生架构运行器。补齐文档要求的基础工具后，调用实际安装函数安装Caddy，检查服务用户与服务文件、验证生产配置；用本地CA测试TLS 1.3、HTTP/2和网页内容，并让本地模拟ACME服务实际访问HTTP-01挑战。
- **3个x86_64 systemd任务**：在Ubuntu 22.04/24.04/26.04临时虚拟机中，实际安装、启动、重载服务；验证重复运行保留自定义网页及同一证书，并运行本地回归测试。
- 容器测试不伪造systemd运行状态；各RPM系统的完整服务启动、SELinux、公网HTTP-01及真实自动续期仍需另测。RHEL暂未加入真实镜像矩阵，不能把其衍生系统的结果算作RHEL实测。
- Debian 11的默认镜像软件源已出现404；CI明确使用Debian官方签名归档源准备该环境，再测试安装。部署脚本不自动改源，因此这项结果只适用于软件源已经可用的Debian 11。
- 所有证书测试使用本地CA，不使用用户域名、账号、VPS或正式证书额度。具体通过/失败以对应提交的Actions结果为准，流程文件存在不代表测试已通过。

发布前审查已修复：确认期间配置变动被覆盖、误认其他同名Caddy进程、前导零端口校验不一致，以及服务配置路径的部分匹配问题。现在会在持锁后重新核对配置与部署记录，并按systemd MainPID确认监听归属。

仓库包含[本地回归测试](tests/reality-site.test.sh)。开发者在Linux上准备Caddy和ShellCheck后可运行：

```bash
bash tests/reality-site.test.sh /path/to/caddy /path/to/shellcheck
```

上述本地回归使用临时目录、本地CA和模拟systemd操作；Actions中的3个systemd任务另行验证真实服务操作。安装中断恢复、RPM系统的完整服务部署、SELinux及公网证书申请/续期仍需进一步验证。

本地已通过 Bash/ShellCheck、系统版本分类、端口冲突、预装 Caddy 退出、重复运行、配置失败恢复，以及真实 Caddy 的 TLS 1.3、HTTP/2 和网页 200。使用本地 CA 的升级测试已验证 HTTP 端口释放及原证书复用。新增系统还覆盖 DNF4/DNF5 插件、EL/Fedora及架构的软件源选择、依赖失败停止；这些安装分支使用模拟包管理器测试。2026年11月18日已读取验证DEB的amd64/arm64包索引，以及EL 8/9/10、Fedora 43/44/45两种架构的全部12份RPM仓库元数据；这只确认对应软件源可用，不代替各系统的依赖解析和完整安装。

旧版曾在 Debian 13/amd64 完成真实安装、HTTP-01 正式签发和本机 HTTPS 验证。2026年11月18日HTTP-01绑定修复后，用户在重装的Debian 13/amd64回传：本机证书受信任、域名匹配、网页200、HTTP/2成功。当前版的自动化测试范围和结果见上面的Actions记录；公网自动续期及RPM系统的systemd/SELinux仍未覆盖。

参考：[Caddy 自动 HTTPS](https://caddyserver.com/docs/automatic-https)、[关闭自动跳转](https://caddyserver.com/docs/caddyfile/options#auto-https)。
