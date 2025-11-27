# AutoPay 人寿支付认证系统（正式版）

基于 Vercel + Supabase 的全静态前端方案，包含：

- `autopay.html`：内部后台生成订单页面
- `verify.html`：客户确认投保信息页面（支持多页投保单轮播）
- `signature.html`：电子签名页面（手写 + 上传图片二合一）
- `pay.html`：支付页面（支付宝二维码 + 微信收款码预览）
- `global.css`：统一 UI 风格
- `vercel.json`：将根路径 `/` 定向到 `autopay.html`

已内置你的 Supabase 配置：

- SUPABASE_URL = https://pefccznphgacswztqzrp.supabase.co
- SUPABASE_ANON_KEY = eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InBlZmNjem5waGdhY3N3enRxenJwIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NjQxNTQwMTgsImV4cCI6MjA3OTczMDAxOH0.nz5gdOBpGzWIRm8qU49O0JFCyMc3csmHwhNA_YkpixY

## 一、部署到 Vercel

1. 登录 Vercel（用 GitHub / GitLab / Email 均可）
2. 新建项目 → 上传本 ZIP 解压后的目录
3. 部署时无需任何 Build 命令（静态站点）
4. 部署完成后，访问：

   - 后台页面（生成订单）：`https://你的域名/autopay.html`
   - 客户确认页：`https://你的域名/verify.html?id=订单ID`
   - 签名页：`https://你的域名/signature.html?id=订单ID`
   - 支付页：`https://你的域名/pay.html?id=订单ID`

> 由于支付网关不提供回调，本系统采用「客户支付完成后点击按钮 → 更新 Supabase 状态」的半自动模式，已经尽量贴近你选择的“自动更新”选项。

## 二、Supabase 需求配置

1. 数据表：`insurance_orders` 至少包含字段：

   - `id` (text, primary key)
   - `plate` (text)
   - `fee` (numeric / double)
   - `period` (text)
   - `policy_images` (text[] / jsonb)
   - `wechat_qr_url` (text)
   - `alipay_qr_url` (text)
   - `signature_url` (text)
   - `status` (text)

2. 存储 bucket：

   - `insure-policy` 用于投保单图片
   - `insure-wechat` 用于微信收款二维码
   - `insure-signature` 用于客户签名图片

3. RLS：需确保 anon 角色具备必要权限（示例）：

   - `SELECT` / `INSERT` / `UPDATE` on `insurance_orders`
   - 存储 bucket 允许 `INSERT`、`UPDATE`、`SELECT` public 对象

请根据你的公司安全规范自行细化策略。

## 三、使用流程（客户视角）

1. 业务员在 `autopay.html` 中生成订单
2. 把 `verify.html?id=...` 链接发给客户
3. 客户确认投保单 → 跳转至签名页
4. 客户完成签名（手写或上传图片）→ 系统写入 Supabase
5. 跳转支付页，客户扫码完成支付
6. 客户点击“我已完成支付”按钮 → 系统将订单状态更新为 `paid`

你可以在 Supabase 控制台或者你自己的运营后台中，实时查看订单状态流转。
