# MoveCar - 挪车通知系统

基于 Cloudflare Workers 的智能挪车通知系统，扫码即可通知车主，保护双方隐私。

## 界面预览

| 请求者页面 | 车主页面 |
|:---:|:---:|
| [🔗 在线预览](https://htmlpreview.github.io/?https://github.com/lesnolie/movecar/blob/main/preview-requester.html) | [🔗 在线预览](https://htmlpreview.github.io/?https://github.com/lesnolie/movecar/blob/main/preview-owner.html) |

## 为什么需要它？

- 🚗 **被堵车却找不到车主** - 干着急没办法
- 📱 **传统挪车码暴露电话** - 隐私泄露、骚扰电话不断
- 😈 **恶意扫码骚扰** - 有人故意反复扫码打扰
- 🤔 **路人好奇扫码** - 并不需要挪车却触发通知

## 这个系统如何解决？

- ✅ **不暴露电话号码** - 通过推送通知联系，保护隐私
- ✅ **双向位置共享** - 车主可确认请求者确实在车旁
- ✅ **无位置延迟 30 秒** - 降低恶意骚扰的动力
- ✅ **免费部署** - Cloudflare Workers 免费额度完全够用
- ✅ **无需服务器** - Serverless 架构，零运维成本

## 为什么使用 Bark 推送？

- 🔔 支持「紧急 / 重要 / 警告」通知级别
- 🎵 可自定义通知音效
- 🌙 **即使开启勿扰模式也能收到提醒**
- 📱 安卓用户：原理相通，将 Bark 替换为安卓推送服务即可（如 Pushplus、Server酱）

## 使用流程

### 请求者（需要挪车的人）

1. 扫描车上的二维码，进入通知页面
2. 填写留言（可选），如「挡住出口了」
3. 允许获取位置（不允许则延迟 30 秒发送）
4. 点击「通知车主」
5. 等待车主确认，可查看车主位置

### 车主

1. 收到 Bark 推送通知
2. 点击通知进入确认页面
3. 查看请求者位置（判断是否真的在车旁）
4. 点击确认，分享自己位置给对方

### 流程图

```
请求者                              车主
  │                                  │
  ├─ 扫码进入页面                     │
  ├─ 填写留言、获取位置                │
  ├─ 点击发送                         │
  │   ├─ 有位置 → 立即推送 ──────────→ 收到通知
  │   └─ 无位置 → 30秒后推送 ────────→ 收到通知
  │                                  │
  ├─ 等待中...                        ├─ 查看请求者位置
  │                                  ├─ 点击确认，分享位置
  │                                  │
  ├─ 收到确认，查看车主位置 ←──────────┤
  │                                  │
  ▼                                  ▼
```

## 部署教程

1.Cloudfalre 部署
第一步：注册 Cloudflare 账号
打开 Cloudflare 官网：https://dash.cloudflare.com/sign-up

输入你的邮箱和密码，完成注册并登录。

第二步：创建 Worker (云端运行环境)
登录后，点击左侧菜单栏的「Workers & Pages」。

点击右侧的「Create」按钮，然后选择「Create Worker」。

在 Name（名称）一栏填入 movecar（或者你喜欢的任何英文字母）。

点击右下角的「Deploy」（部署）按钮。

部署完成后，点击「Edit code」（编辑代码），你会看到一个代码编辑器。

将里面的默认代码全部删除。

复制全部代码（博客代码是在原有的 Bark、增加适合国内微信生态的 PushPlus，以及极其稳定且适合极客的 Telegram Bot。），粘贴到刚才清空的编辑器中。

点击右上角的「Deploy」保存代码。

具体代码地址：[https://github.com/illria/movecar/blob/patch-1/movecar.js](https://raw.githubusercontent.com/xinlingmiyu-debug/movecar/refs/heads/main/%E4%BF%AE%E6%94%B9%E8%BF%87movecar.js)

第三步：创建 KV 存储 (云端数据库)
我们需要一个地方来临时存储挪车状态和位置信息。

回到 Cloudflare 左侧菜单，点击「KV」。

点击「Create a namespace」（创建命名空间）。

名称必须填入：MOVE_CAR_STATUS，然后点击「Add」。

重新回到左侧菜单的「Workers & Pages」，点击你刚才创建的 movecar 项目。

进入顶部的「Settings」（设置）选项卡，在左侧找到「Bindings」（绑定）。

点击「Add」-> 选择「KV Namespace」。

Variable name（变量名） 必须准确填入：MOVE_CAR_STATUS。

在下拉菜单中选择你刚刚创建的命名空间，点击「Deploy」保存。

第四步：配置推送环境变量
这一步决定了通知怎么发到你的手机上。

依然在 Worker 的「Settings」中，找到「Variables and Secrets」（变量和机密）。

点击「Add」添加以下变量（任选其中几个通知方式）：

1.bark 通知

变量名: BARK_URL

值: 填入你的 Bark 推送地址（例如 https://api.day.app/你的专属Key）

2. 微信 pushplus 通知

变量名: PUSHPLUS_TOKEN

值: 填入你的 PUSHPLUS_TOKEN 的 token 即可

3.telegram bot 通知

变量名: TG_BOT_TOKEN

变量名:TG_CHAT_ID

值: 填入你的 TOKEN和ID 的即可
### 第五步：绑定域名（可选）

1. Worker →「Settings」→「Domains & Routes」
2. 点击「Add」→「Custom Domain」
3. 输入你的域名，按提示完成 DNS 配置

## 制作挪车码

### 生成二维码

1. 复制你的 Worker 地址（如 `https://movecar.你的账号.workers.dev`）
2. 使用任意二维码生成工具（如 草料二维码、QR Code Generator）
3. 将链接转换为二维码并下载

### 美化挪车牌

使用 AI 工具生成精美的装饰设计：

- **Nanobanana Pro** - 生成装饰图案和背景
- **ChatGPT** - 生成创意设计图

制作步骤：

1. 用 AI 工具生成你喜欢的装饰图案
2. 将二维码与生成的图案组合排版
3. 添加「扫码通知车主」提示文字
4. 打印、过塑，贴在车上

> 💡 用 AI 生成独一无二的挪车牌，让你的爱车更有个性！

### 效果展示

![挪车码效果](demo.jpg)

## 安全设置（推荐）

为防止境外恶意攻击，建议只允许中国地区访问：

### 方法一：使用 WAF 规则（推荐）

1. 进入 Cloudflare Dashboard → 你的域名
2. 左侧菜单点击「Security」→「WAF」
3. 点击「Create rule」
4. 规则设置：
   - Rule name：`Block non-CN traffic`
   - If incoming requests match：`Country does not equal China`
   - Then：`Block`
5. 点击「Deploy」

### 方法二：在 Worker 代码中过滤

在 `movecar.js` 的 `handleRequest` 函数开头添加：

```javascript
async function handleRequest(request) {
  const country = request.cf?.country;
  if (country && country !== 'CN') {
    return new Response('Access Denied', { status: 403 });
  }

  // 下面保持原有逻辑
}
```

> ⚠️ 曾经被境外流量攻击过，强烈建议开启地区限制！

## License

MIT



