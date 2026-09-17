# frontend-rebuild-skills

两套配套 Agent Skill：

1. **public-frontend-rebuild** — 从线上站点已经公开下发的 HTML/CSS/JS/资源，重建一份可读、可点的本地预览。不是克隆私有仓库。
2. **graft-rebuilt-ui** — 把这份本地预览接到自己的产品页上：主区原样挂上、旧 UI 藏起来、只接已有接口。

先 rebuild，再 graft。不要在 rebuild 过程中改自己的产品仓。

仓库是 **private**。别人要装，先被邀请成 collaborator，再走下面的命令。

## 安装

对方需要：

- GitHub 账号已被加进本仓库（Settings → Collaborators）
- 本机 `gh auth login` 已登录（`npx skills` 拉私有仓走 GitHub 凭证）

一次装两个：

```bash
npx skills add kk7041/frontend-rebuild-skills --all
```

只要其中一个：

```bash
npx skills add kk7041/frontend-rebuild-skills --skill public-frontend-rebuild
npx skills add kk7041/frontend-rebuild-skills --skill graft-rebuilt-ui
```

指定 agent（例如 Claude Code / Cursor / Codex）：

```bash
npx skills add kk7041/frontend-rebuild-skills --all --agent claude-code
```

装完后，把线上 URL 丢给 agent，说「扒下来 / 还原前端 / 本地跑这个站」会走 rebuild；指着本地预览说「接到我们的页面 / 移植 UI」会走 graft。

更新：

```bash
npx skills update
```

## 仓库结构

```
public-frontend-rebuild/
  SKILL.md
  references/
    bundle-map.md      # Next / Vite / Modern.js / Cloudflare 等包图
    local-preview.md   # 本地 Next 预览端口与代理坑
graft-rebuilt-ui/
  SKILL.md
  references/
    shell-fit.md       # dashboard 壳层 padding / 100vh / token 作用域
```

## 边界

- 只读浏览器本来就会下的公开包。
- 不碰登录、私有 API、支付、绕过付费墙。
- 结果是重建预览，不是原仓库源码。原站品牌、文案、资源仍属原作者。
- graft 只接自己产品里已经有的接口；缺后端的菜单只提示，不编造假 API。

## 邀请别人

```bash
gh api -X PUT repos/kk7041/frontend-rebuild-skills/collaborators/<github-username> -f permission=pull
```

对方接受邀请后即可 `npx skills add`。
