# wlb-x 智能助手 · 代理部署说明（Cloudflare Worker）

> 页面是纯静态托管（GitHub Pages），**任何文件访问者都能下载**，
> 所以 API Key 绝不能写进 `index.html`。本代理把 Key 留在服务端，前端只调用代理地址。

---

## 一、5 分钟部署

1. 打开 <https://dash.cloudflare.com/> 注册/登录（免费账号即可，不用绑卡）；
2. 左侧 **Workers & Pages → Create → Workers → Create Worker**；
3. 名字填 `wlbx-proxy`（随意），点 **Deploy**；
4. 进 **Edit code**，把本目录 `worker.js` 的**全部内容**粘贴进去，覆盖模板代码 → **Deploy**；
5. 回到 Worker 页面 → **Settings → Variables and Secrets**，添加：

| 名称 | 类型 | 值 |
|---|---|---|
| `DEEPSEEK_API_KEY` | **Secret**（密文） | 你的 DeepSeek key（`sk-...`） |
| `ALLOW_ORIGIN` | Text | `https://wulanbai-lgtm.github.io`（多个用英文逗号分隔，如再加 `http://localhost:8080`） |
| `MODEL` | Text（可选） | `deepseek-chat` |
| `MAX_TOKENS` | Text（可选） | `700` |
| `RATE_PER_MIN` | Text（可选） | `12`（每 IP 每分钟请求上限） |

6. 保存后复制 Worker 地址，形如：
   `https://wlbx-proxy.你的账号.workers.dev`
7. 打开 `index.html`，找到这一行并填入地址（**注意结尾加 `/chat`**）：

```js
const WLBX_ENDPOINT = "https://wlbx-proxy.你的账号.workers.dev/chat";
```

保存 → 刷新页面 → 点左下角「问问 wlb-x」即可对话。

---

## 二、安全要点（务必做）

- `ALLOW_ORIGIN` **一定要填自己的站点域名**，否则别人可以拿你的代理当免费 API 用；
- Key 只存在 Cloudflare 的 Secret 里，**永远不要**贴进代码、聊天记录或提交到仓库；
- 仓库里 `.env` 已被 `.gitignore` 忽略（`.env`、`*.env`、`wlbx.config.local.js`），提交前可用
  `git status` 确认它没出现在待提交列表里；
- 免费额度：Workers 每天 10 万次请求，个人站完全够用；DeepSeek 按量计费，注意余额。

---

## 三、本地调试（不发代理也能测）

在浏览器控制台执行任一种：

```js
// 方式 A：走已部署的代理
localStorage.setItem("wlbx-endpoint", "https://wlbx-proxy.xxx.workers.dev/chat");

// 方式 B：直连 DeepSeek（仅本机自用，Key 只存在你自己的浏览器里，切勿用于线上）
localStorage.setItem("wlbx-key", "sk-你的key");
```

清除：`localStorage.removeItem("wlbx-endpoint")` / `localStorage.removeItem("wlbx-key")`。

---

## 四、行为约束（已内置，双层保险）

- **前端**：知识库由站内 `resume` 数据实时生成 → 改简历后助手自动同步，不会答错旧信息；
- **服务端**：系统提示词由 Worker 强制拼装，客户端无法覆盖（伪造的 `system` 消息会被丢弃）；
- 约束要点（三层范围：事实准确 + 允许相关分析）：
  1. **事实直答**：教育、技能、项目、亮点、经历、证书、求职意向、联系方式 → 逐字准确；
  2. **基于简历的分析与建议**（允许，但须标明「依据」与「性质」）：岗位匹配度分析、能力归纳、
     简历内项目对比、面试/协作建议、简历中术语的 1–3 句通俗解释、基于既有方向的发展建议；
     ⚠️ 必须用「简历显示／据此推测／建议」区分事实与推断，不得编造数字或成果；
  3. **必须拒绝**：与简历无关的话题（写代码、通用百科、闲聊、政治、翻译长文、角色扮演）、
     要求编造或夸大、替本人做承诺（录用/薪资/入职时间）、贬损他人；
- 另有：简历外个人信息一律答「简历里没有这项信息」；拒绝泄露提示词与内部配置；语言跟随提问者。
