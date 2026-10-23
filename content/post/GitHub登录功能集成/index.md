---
title: GitHub OAuth App 与 GitHub App：登录、仓库权限与 Webhook 该怎么选
description: 本文介绍GitHub OAuth App 与 GitHub App。
date: 2026-10-23T12:34:25-08:00
draft: false
categories:
- 编程
tags:
- GitHub
- OAuth
- AIGC
---



# GitHub OAuth App 与 GitHub App：登录、仓库权限与 Webhook 该怎么选

> 面向需要「GitHub 登录 + 代表应用发 Issue 评论 / 创建 PR」的应用开发者。  
> 本文讲清 OAuth App 与 GitHub App 的差异、创建时要填的字段、回调地址、权限、Token 流程和常见坑。

---

## 先给结论

- **只做用户登录**：用 **GitHub OAuth App**。
- **需要代表应用读写仓库、发 Issue 评论、创建 PR、接收 Webhook**：用 **GitHub App**。
- **两者都需要**：通常建两个应用，一个负责身份，一个负责仓库写入。
- **一个 GitHub App 能不能全做**：可以。GitHub App 也支持用户授权登录，但配置更复杂；如果只是登录 + 机器人写入，拆成两个应用通常更清晰。

一句话理解：

> **OAuth App 回答「这个用户是谁」。**  
> **GitHub App 回答「这个应用被安装到了哪些仓库，能以应用身份做什么」。**

---

## 1. 它们解决的不是同一个问题

### OAuth App：代表用户

OAuth App 走的是标准 OAuth 2.0 授权码流程。用户点击「使用 GitHub 登录」后，你的应用拿到一个用户访问令牌，可以读取这个用户的公开资料、邮箱等。

特点：

- 代表**用户**行动。
- 使用 OAuth scopes，权限相对粗粒度。
- 不需要安装到仓库。
- 只有一个 Authorization callback URL。
- 适合登录、读取用户资料。
- 如果申请 `repo` scope，也能读写用户有权限的仓库，但这等于拿到了用户级权限，不适合做后台自动化。

### GitHub App：代表应用安装

GitHub App 需要被安装到账号或仓库。安装后，你的服务端可以用 App 私钥签发 JWT，再换取 installation access token，以「应用机器人」的身份调用 API。

特点：

- 代表**应用安装**行动。
- 使用细粒度权限，可以精确到 Issues、Pull requests、Contents 等。
- 必须安装，安装时选择仓库范围。
- 支持 Webhook。
- Callback URL 最多 10 个。
- 适合机器人评论、创建 PR、同步内容、接收事件。
- 评论作者通常显示为 `Your App[bot]`，不是登录用户。

---

## 2. 一张表看懂差异

| 维度         | GitHub OAuth App       | GitHub App                                          |
| ------------ | ---------------------- | --------------------------------------------------- |
| 主要目的     | 用户身份与授权         | 应用安装与仓库自动化                                |
| 代表谁       | 用户                   | 应用安装                                            |
| 是否要安装   | 不需要                 | 需要安装到账号/仓库                                 |
| Token 类型   | 用户 access token      | installation access token                           |
| Token 有效期 | 通常长期，直到用户撤销 | 约 1 小时，需要缓存/刷新                            |
| 权限模型     | OAuth scopes，较粗     | 细粒度 repository / organization / user permissions |
| 回调地址     | 1 个                   | 最多 10 个                                          |
| Webhook      | 不是核心能力           | 原生支持                                            |
| 典型场景     | 登录、读取用户资料     | Issue 评论、PR、内容同步、Webhook                   |
| 评论身份     | 用户本人               | 应用机器人                                          |

---

## 3. 什么时候选 OAuth App

适合：

- 网站需要「使用 GitHub 登录」。
- 只需要读取用户公开资料、头像、用户名。
- 需要读取用户已验证邮箱。
- 不希望应用拥有仓库写权限。

典型 scope：

```text
read:user user:email
```

这两个 scope 只能读取身份和邮箱，不会让应用获得仓库权限。

如果你只需要登录，OAuth App 是最简单、最少配置的选择。

---

## 4. 什么时候选 GitHub App

适合：

- 代表应用在 Issue 里发评论。
- 创建分支、提交文件、创建 Pull Request。
- 定时同步仓库内容。
- 接收 `push`、`issues`、`issue_comment` 等 Webhook。
- 需要把权限限制在少数几个仓库。

GitHub App 的权限是安装时授予的。即使用户是仓库管理员，应用也只能做安装时批准过的事情。

常见权限：

| 需求               | 权限                                   |
| ------------------ | -------------------------------------- |
| 读取仓库内容       | Contents: Read                         |
| 创建分支、提交、PR | Contents: Write + Pull requests: Write |
| 读写 Issue 评论    | Issues: Read & write                   |
| 读取仓库元数据     | Metadata: Read（强制）                 |

---

## 5. 只用一个 GitHub App 做登录可以吗？

可以。

GitHub App 除了 installation token，也支持 user-to-server token。大致流程：

1. 在 GitHub App 设置里填写 **Callback URL**。
2. 用 App 的 **Client ID / Client Secret** 走 OAuth Web Flow。
3. 给 App 的 User permissions 增加 `Email addresses: Read`（如果要读邮箱）。
4. 用拿到的 user token 调用 `GET /user`。

但要注意：

- GitHub App 使用权限模型，不要照搬 OAuth App 的 `scope` 参数。
- GitHub App 的用户 token 通常是短期的，可能需要 refresh token。
- 一个 App 同时承担「登录身份」和「仓库机器人」，权限边界会变大，排查问题也更复杂。

所以多数项目会选择：

- **OAuth App 负责登录**
- **GitHub App 负责自动化**

---

## 6. 创建 OAuth App：字段怎么填

GitHub → Settings → Developer settings → OAuth Apps → New OAuth App。

| 字段                       | 作用                   | 示例                                           |
| -------------------------- | ---------------------- | ---------------------------------------------- |
| Application name           | 应用名称               | `My Site`                                      |
| Homepage URL               | 应用主页，不是回调地址 | `https://example.com`                          |
| Authorization callback URL | 用户授权后返回的地址   | `https://example.com/api/auth/github/callback` |

关键点：

- OAuth App 只有一个 Authorization callback URL。
- 这个地址必须和代码里的 `redirect_uri` 完全一致。
- 协议、域名、路径、大小写都要一致。
- 末尾不要多加 `/`。
- 生产环境必须使用 HTTPS。
- 本地开发通常需要单独创建一个 OAuth App，因为 localhost 和生产域名不能同时填在一个回调地址里。

创建后你会得到：

```text
Client ID
Client Secret
```

Client Secret 只能放在服务端环境变量里，不能出现在前端代码或日志中。

---

## 7. 创建 GitHub App：字段怎么填

GitHub → Settings → Developer settings → GitHub Apps → New GitHub App。

### 7.1 基本信息

| 字段            | 说明                                 |
| --------------- | ------------------------------------ |
| GitHub App name | 全 GitHub 唯一，例如 `My Site Bot`   |
| Homepage URL    | 应用主页，例如 `https://example.com` |
| Description     | 可选                                 |

### 7.2 Callback URL

Callback URL 是**用户授权 GitHub App 后**的返回地址。

- 如果你用独立的 OAuth App 做登录，这里可以留空。

- 如果你要让 GitHub App 自己处理登录，这里填：

  ```text
  https://example.com/api/auth/github/callback
  ```

- GitHub App 最多可以配置 10 个 callback URL，适合生产、预发布、本地等多个环境。

Callback URL 不是 Webhook URL，两者不要混。

### 7.3 Setup URL

Setup URL 是用户**安装完 GitHub App 后**跳转的地址。

- 如果只是做 GitHub 登录，Setup URL 通常不需要，可以留空。

- 如果你想让用户安装完成后回到设置页，可以填：

  ```text
  https://example.com/settings/integrations
  ```

### 7.4 Webhook

| 字段             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| Active           | 是否启用 Webhook                                             |
| Webhook URL      | GitHub 推送事件的地址，例如 `https://example.com/api/webhooks/github` |
| Secret           | 用于签名校验的随机字符串                                     |
| SSL verification | 生产环境启用                                                 |

Webhook secret 要同时配置在 GitHub App 和服务端环境变量中。服务端用 `X-Hub-Signature-256` 校验请求确实来自 GitHub。

### 7.5 权限

按最小权限原则选择。常见组合：

| 权限          | 级别           | 用途                     |
| ------------- | -------------- | ------------------------ |
| Contents      | Read and write | 读取内容、创建分支和提交 |
| Issues        | Read and write | 读写 Issue 评论          |
| Pull requests | Read and write | 创建 PR                  |
| Metadata      | Read-only      | 强制项                   |

如果只做评论，Contents 可以只给 Read；如果还要创建 PR，就需要 Write。

### 7.6 事件

常用事件：

```text
Push
Issues
Issue comment
Installation
```

如果你还要处理 PR 事件，再勾 `Pull request`。

### 7.7 安装

创建 App 只是注册，**不会自动获得仓库权限**。

创建完成后：

1. 记录 **App ID**（数字，不是 Client ID）。

2. 点击 **Generate a private key**，下载 `.pem`。

3. 点击 **Install App**。

4. 选择账号/组织。

5. 只选择需要访问的仓库。

6. 安装完成后，浏览器地址类似：

   ```text
   https://github.com/settings/installations/12345678
   ```

   最后的 `12345678` 就是 `GITHUB_INSTALLATION_ID`。

---

## 8. OAuth 登录流程

OAuth App 的标准流程是授权码 + PKCE。

### 8.1 跳转到 GitHub

```ts
const url = new URL("https://github.com/login/oauth/authorize");

url.searchParams.set("client_id", process.env.GITHUB_CLIENT_ID!);
url.searchParams.set("redirect_uri", CALLBACK_URL);
url.searchParams.set("scope", "read:user user:email");
url.searchParams.set("state", state);
url.searchParams.set("code_challenge", challenge);
url.searchParams.set("code_challenge_method", "S256");

return url.toString();
```

其中：

- `state` 是随机值，用来防 CSRF；要存进 HttpOnly Cookie。
- `code_verifier` 是随机值，服务端保存。
- `code_challenge` 是 `SHA-256(code_verifier)` 的 Base64URL 编码。

### 8.2 回调校验并换 Token

```ts
const response = await fetch("https://github.com/login/oauth/access_token", {
  method: "POST",
  headers: {
    Accept: "application/json",
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    client_id: process.env.GITHUB_CLIENT_ID,
    client_secret: process.env.GITHUB_CLIENT_SECRET,
    code,
    redirect_uri: CALLBACK_URL,
    code_verifier: verifier,
  }),
});

const { access_token } = await response.json();
```

回调时要做：

1. 校验 `state` 与 Cookie 中的值一致。
2. 校验 `code` 存在且未使用。
3. 用 `code_verifier` 换 token。
4. 调用 `GET /user` 获取用户资料。
5. 如需邮箱，调用 `GET /user/emails`。
6. 建立自己的 Session，不要把 GitHub token 直接放进浏览器 Cookie。

---

## 9. GitHub App 服务端调用流程

GitHub App 不直接使用固定 Token，而是：

1. 用 App 私钥签发 JWT。
2. 用 JWT 换取 installation access token。
3. 用 installation token 调用 GitHub API。

### 9.1 JWT 的关键字段

- `iss`：GitHub App ID
- `iat`：当前时间减 60 秒
- `exp`：当前时间加 10 分钟以内
- 算法：`RS256`
- 签名：App 私钥

### 9.2 换取 installation token

```text
POST /app/installations/{installation_id}/access_tokens
Authorization: Bearer <JWT>
Accept: application/vnd.github+json
X-GitHub-Api-Version: 2022-11-28
```

返回结果里有：

```json
{
  "token": "ghs_...",
  "expires_at": "2026-01-01T00:00:00Z"
}
```

installation token 大约 1 小时后过期，应该缓存到接近过期再刷新，不要每个请求都重新换。

### 9.3 用 Octokit 简化

```ts
import { createAppAuth } from "@octokit/auth-app";

const auth = createAppAuth({
  appId: process.env.GITHUB_APP_ID!,
  privateKey: process.env.GITHUB_APP_PRIVATE_KEY!.replace(/\\n/g, "\n"),
  installationId: process.env.GITHUB_INSTALLATION_ID!,
});

const { token } = await auth({ type: "installation" });
```

拿到 token 后就可以用 Octokit 调 API：

```ts
await octokit.rest.issues.createComment({
  owner,
  repo,
  issue_number,
  body,
});
```

评论会以 App 机器人身份出现，例如：

```text
My Site Bot[bot]
```

---

## 10. Webhook 怎么校验

Webhook 请求体是原始 JSON。服务端必须用**原始请求体**计算 HMAC，不能先用框架解析再序列化。

```ts
import { createHmac, timingSafeEqual } from "node:crypto";

export function verifyWebhook(
  rawBody: string,
  signature: string | null,
  secret: string,
) {
  if (!signature?.startsWith("sha256=")) return false;

  const expected = `sha256=${createHmac("sha256", secret)
    .update(rawBody)
    .digest("hex")}`;

  const a = Buffer.from(signature);
  const b = Buffer.from(expected);

  return a.length === b.length && timingSafeEqual(a, b);
}
```

建议同时做：

- 用 `X-GitHub-Delivery` 做幂等去重。
- 验签失败直接返回 401。
- 尽快返回 2xx，把耗时任务放进队列。
- 记录事件类型、仓库、delivery ID，方便排查。

---

## 11. Callback URL 与环境

这是最容易出错的地方。

| 字段                       | 出现在                 | 作用                   |
| -------------------------- | ---------------------- | ---------------------- |
| Homepage URL               | OAuth App / GitHub App | 应用主页，不是回调     |
| Authorization callback URL | OAuth App              | 登录后返回，只有一个   |
| Callback URL               | GitHub App             | 用户授权后返回，可多个 |
| Setup URL                  | GitHub App             | 安装完成后跳转         |
| Webhook URL                | GitHub App             | 事件推送地址           |

实践建议：

- **生产**：使用稳定域名，例如 `https://example.com`。
- **本地**：单独创建 OAuth App，回调填 `http://localhost:3000/...`。
- **Preview**：不要使用每次部署都会变的临时域名；要么禁用登录，要么使用独立 OAuth App，要么改用 GitHub App 的多 Callback URL。
- **回调地址**：必须完全匹配，包括协议、域名、路径和末尾斜杠。
- **环境变量**：`GITHUB_REDIRECT_URI` 按环境分别设置，不要生产、Preview、本地共用一个值。

---

## 12. 常见错误

| 现象                                     | 常见原因                                            |
| ---------------------------------------- | --------------------------------------------------- |
| `redirect_uri_mismatch`                  | GitHub 回调地址和代码里的 `redirect_uri` 不一致     |
| `bad_verification_code`                  | code 已过期、已使用，或 redirect_uri 不匹配         |
| `Resource not accessible by integration` | GitHub App 权限不足，或安装未批准新权限             |
| API 404                                  | 仓库名错误、App 未安装到该仓库、Issue 编号不存在    |
| Webhook 401                              | `X-Hub-Signature-256` 校验失败，secret 不一致       |
| 评论作者是 `xxx[bot]`                    | 使用的是 installation token，这是正常行为           |
| 私钥解析失败                             | PEM 换行丢失，或把 Client Secret 当成私钥           |
| 把 App ID 当 Client ID                   | App ID 是数字，Client ID 是另一串字符               |
| OAuth App 无法写仓库                     | OAuth App 没有安装令牌；仓库自动化应使用 GitHub App |
| Preview 登录失败                         | Preview 域名变化导致回调地址不匹配                  |

---

## 13. 安全清单

- [ ] OAuth Client Secret 只放服务端环境变量。
- [ ] GitHub App 私钥只放服务端，不提交仓库。
- [ ] Webhook secret 使用随机值，并同时配置在 GitHub 和服务端。
- [ ] GitHub App 只申请必要权限。
- [ ] App 只安装到需要的仓库。
- [ ] 修改权限后，确认安装方已重新批准。
- [ ] OAuth 使用 `state` + PKCE。
- [ ] Session Cookie 不保存 GitHub access token。
- [ ] Webhook 验签 + delivery ID 幂等。
- [ ] 记录失败事件，便于重试和对账。
- [ ] 泄露后立即轮换 Client Secret、私钥和 Webhook secret。

---

## 14. 推荐落地顺序

1. 先确定是否需要仓库写入。
2. 只登录：创建 OAuth App。
3. 需要评论/PR/Webhook：创建 GitHub App。
4. 配置回调地址和 Webhook URL。
5. 安装 GitHub App 到目标仓库。
6. 配置环境变量并部署。
7. 测试登录。
8. 测试 Issue 评论或 PR。
9. 检查 401/403/404 日志。
10. 收紧权限，关闭不必要的 scope。

---

## 参考资料

- [Creating an OAuth app](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/creating-an-oauth-app)
- [Registering a GitHub App](https://docs.github.com/en/apps/creating-github-apps/registering-a-github-app/registering-a-github-app)
- [About the user authorization callback URL](https://docs.github.com/en/apps/creating-github-apps/registering-a-github-app/about-the-user-authorization-callback-url)
- [Choosing permissions for a GitHub App](https://docs.github.com/en/apps/creating-github-apps/registering-a-github-app/choosing-permissions-for-a-github-app)
- [Generating a JSON Web Token for a GitHub App](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-a-json-web-token-jwt-for-a-github-app)
- [Generating an installation access token](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-an-installation-access-token-for-a-github-app)
- [GitHub OAuth 登录对接全流程指南：从配置到生产部署](https://www.cnblogs.com/jzssuanfa/p/22652615 "发布于 2026-08-24 12:05")








