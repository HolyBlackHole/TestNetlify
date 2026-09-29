# Netlify 部署测试站

一个零依赖的纯静态页面，用来验证「GitHub 仓库 → Netlify 自动部署」这条链路是否打通。

仓库里原来的 `main.py` 是 PyCharm 的示例脚本，**Netlify 不托管 Python 脚本**，所以需要这样一份真正的静态页面才能部署出可访问的网站。

## 目录结构

```
netlify测试站点/
├── netlify.toml          # Netlify 配置：发布目录 = public
├── .gitignore            # 排除 .idea、__pycache__ 等不该提交的文件
└── public/
    ├── index.html        # 首页（含版本号自检面板）
    └── 404.html          # 自定义 404 页面
```

## 一、把文件放进你的仓库

在你克隆下来的仓库目录里执行（把路径换成你实际存放的位置）：

```bash
git clone https://github.com/HolyBlackHole/TestNetlify.git
cd TestNetlify

# 把 netlify.toml、.gitignore、public/ 拷进仓库根目录
# 顺手清理掉不该提交的 IDE 文件
git rm -r --cached .idea

git add .
git commit -m "add static site for netlify deploy test"
git push
```

> 建议同时在 GitHub 仓库页面 Settings → General → 把默认分支设为你实际使用的分支（当前仓库默认是 `master`）。

## 二、在 Netlify 上连接仓库

1. 打开 https://app.netlify.com ，用 GitHub 账号登录（Authorize 时至少勾选这个仓库的权限）。
2. 点击 **Add new site → Import an existing project → GitHub**，选中 `HolyBlackHole/TestNetlify`。
3. 构建设置按下面填：

| 配置项 | 填写内容 | 说明 |
| --- | --- | --- |
| Branch to deploy | `master` | 你仓库的默认分支 |
| Build command | 留空 | 纯静态页面，没有构建步骤 |
| Publish directory | `public` | 和 `netlify.toml` 里的 `publish` 保持一致 |

4. 点 **Deploy site**，等状态变成绿色的 Published，Netlify 会分配一个形如 `https://xxxx.netlify.app` 的地址。

> 如果仓库列表里看不到 `TestNetlify`，去 GitHub 的 **Settings → Applications → Netlify** 里给这个仓库授权。

## 三、验证自动部署真的生效

1. 打开 `public/index.html`，把这一行里的版本号改掉：

   ```js
   var SITE_VERSION = "v1-2026-09-29";   // 改成 "v2" 之类
   ```

2. `git commit -m "test deploy v2" && git push`
3. 回到 Netlify 的 **Deploys** 页面，能看到一条新的构建记录；构建完成后刷新线上地址，页面上的「站点版本」变成 `v2`，说明 CD 链路已打通。

首页的「部署自检」面板会显示站点版本、页面加载时间、当前域名和协议，方便你每次部署后快速确认是不是最新版本。

## 四、不想连 Git 时的两种快速验证方式

- **拖拽部署**：打开 https://app.netlify.com/drop ，把 `public` 文件夹（或它的 zip 包）直接拖进去，几秒就能出链接。注意 zip 必须是扁平结构，`index.html` 要在压缩包最外层。
- **CLI 部署**：

  ```bash
  npm install -g netlify-cli
  netlify login
  netlify deploy --dir=public --prod
  ```

## 五、常见问题

- **打开是 Netlify 默认欢迎页 / Page not found**：发布目录里没找到 `index.html`，检查 Publish directory 是否填成 `public`（不要写成 `/public`，也不要带前导斜杠）。
- **改代码后线上没变化**：确认推送的是 Netlify 里配置的那个分支，并去 Deploys 页面看最新一次构建是否成功。
- **样式或图片 404**：检查 HTML 里的资源路径是否和发布目录结构对得上；Netlify 运行在 Linux 上，文件名区分大小写。
- **国内访问偏慢**：Netlify 边缘节点在亚洲覆盖有限，若面向国内用户，可考虑 Cloudflare Pages 作为替代或加一层 CDN。
- **免费额度**：Netlify 免费版为积分制（约 300 积分/月，一次生产部署约 15 积分），个人测试站点完全够用。
