# 抢手的哥的技术博客

> 不会敲代码的老司机不是好厨子

基于 [Hexo](https://hexo.io/) 构建的静态技术博客，主要记录 iOS / Swift / 前端 / 工具链相关的实践与踩坑经验。

**线上地址**：https://blog.95783669.xyz

---

## 技术栈

| 层次 | 方案 |
|---|---|
| 静态站点生成 | Hexo 8.1.2（Node ≥ 18） |
| 主题 | [NexT](https://theme-next.js.org/) v8.29.1（Gemini 方案） |
| 托管 | GitHub Pages（`CoderHRXu/CoderHRXu.github.io`，`master` 分支） |
| DNS / CDN | Cloudflare（代理模式，隐藏源站） |
| 渲染 | EJS + Stylus + Marked |

## 目录结构

```
hexoblog/
├── _config.yml          # 站点配置（标题、URL、permalink、deploy 等）
├── _config.next.yml     # NexT 主题自定义配置（scheme、菜单、搜索等）
├── scaffolds/           # 新建文章/页面模板
├── source/
│   ├── CNAME            # 自定义域名 → blog.95783669.xyz
│   ├── _posts/          # 博客文章（Markdown，24 篇）
│   ├── categories/      # 分类页
│   └── tags/            # 标签页
├── themes/
│   ├── next/            # NexT 主题（git clone，勿直接改其 _config.yml）
│   └── landscape/       # 旧主题（备用）
├── public/              # 构建产物（generate 后生成，不入库）
└── package.json         # 依赖与 Hexo 版本声明
```

## 本地开发

```bash
# 1. 安装依赖
npm install

# 2. 启动本地预览服务（默认 http://localhost:4000）
npx hexo server

# 或指定端口
npx hexo server -p 5000
```

## 写作与发布

```bash
# 新建文章（生成到 source/_posts/）
npx hexo new "文章标题"

# 新建独立页面
npx hexo new page "about"

# 清理 → 构建 → 部署（一条命令走完）
npx hexo clean && npx hexo generate && npx hexo deploy
```

部署走 `hexo-deployer-git`，推送到 `git@github.com:CoderHRXu/CoderHRXu.github.io.git` 的 `master` 分支，由 GitHub Pages 自动发布。

### 部署注意事项

0. **Hexo 已升级到 8.x**：原 3.4.4 在 Node 22 上会静默生成 0 字节空文件（表现为整站空白）。若看到 `public/` 里文件都是 0 字节，先跑 `npx hexo generate --force`。历史依赖已备份在 `package.json.bak`。
1. **CNAME 必须存在且内容正确**：`source/CNAME` 内容为 `blog.95783669.xyz`。构建后请确认 `public/CNAME` 非空——曾有 Hexo 生成空 CNAME 文件导致 GitHub Pages 不认自定义域名、整站 404 的情况。
2. **SSH 认证**：本机使用 `~/.ssh/id_ed25519` 推送，公钥已添加到 GitHub 账号。若更换机器，需重新配置。
3. **代理干扰**：若所在环境设置了 `HTTP_PROXY` / `HTTPS_PROXY`，代理可能拦截 `github.com` 的 CONNECT 请求导致推送失败（502）。可对 git 临时绕过：

   ```bash
   git -c http.proxy= -c https.proxy= push
   ```

4. **DNS 侧**：Cloudflare 上 `blog.95783669.xyz` 需指向 GitHub Pages（A 记录 / CNAME），并保持代理开启。切换自定义域名后，GitHub Pages 的 TLS 证书需几分钟重新签发，期间可能短暂 404 或证书告警。

## 文章分类概览

| 方向 | 代表文章 |
|---|---|
| iOS 适配 | 《AdaptionForiOS11andiPhoneX》《iPhoneXNetworkTypeIssue》 |
| Swift 实践 | 《MethodsOfSwift》《Swift4QRCodeTool》《SwiftURLEncodeing》《ContainInSwift》 |
| Objective-C | 《MethodsOfObjective-C》《iOS_Coding_Guidelines》 |
| 构建与工具链 | 《FastlaneWithXcode9Issue》《IncreasBuildNumberWithFastlane》《UseShellModifyXcodeBuildNumber》《updateCocoapodsIssue》《Gitlab-RunnerErrorAnalysis》《Xcode10-NewBuilSystemIssue》 |
| 网络与服务 | 《KoaStaticServer》《NetEaswIMReceiption》《SelfSginedCertificateAdaptionForMoyaAndKingfisher》 |
| 效率 / 其他 | 《Bus_JSBox》《MacOSShareWifi》《Distance》《JsonToModelPit》 |

## 配置速查（`_config.yml`）

| 键 | 当前值 | 说明 |
|---|---|---|
| `title` | 抢手的哥的技术博客 | 站点标题 |
| `url` | https://blog.95783669.xyz | **必须与自定义域名一致**，影响站点地图与绝对链接 |
| `permalink` | `:year/:month/:day/:title/` | 文章永久链接格式 |
| `language` | zh-CN | 中文（与 NexT 语言包对应） |
| `theme` | next | 主题名需与 `themes/` 下目录名一致，否则整站渲染空白 |
| `deploy.repo` | `git@github.com:CoderHRXu/CoderHRXu.github.io.git` | 部署目标仓库 |
| `deploy.branch` | master | 部署分支 |

---

## 许可

文章内容版权归作者所有，转载请注明出处。站点代码部分可参考 [Hexo](https://github.com/hexojs/hexo) 的开源许可。
