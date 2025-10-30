# 贪吃蛇游戏 - 腾讯云部署指南

## 📦 项目文件清单

本项目包含以下文件：
- `snake_game.html` - 完整的贪吃蛇游戏（单文件，包含所有HTML、CSS和JavaScript）
- `DEPLOYMENT.md` - 本部署指南
- `README.md` - 项目说明文档

## 🚀 腾讯云部署方案

### 方案一：使用腾讯云对象存储（COS）- 推荐方案

这是最简单和经济的部署方式，适合静态HTML文件。

#### 步骤：

1. **登录腾讯云控制台**
   - 访问 https://console.cloud.tencent.com/
   - 登录您的腾讯云账号

2. **开通对象存储服务（COS）**
   - 在控制台搜索"对象存储"或访问 https://console.cloud.tencent.com/cos
   - 点击"立即开通"（如果未开通）

3. **创建存储桶**
   - 点击"存储桶列表" > "创建存储桶"
   - 名称：自定义（如：snake-game-bucket）
   - 所属地域：选择离您用户最近的地域
   - 访问权限：选择"公有读私有写"
   - 点击"确定"创建

4. **上传游戏文件**
   - 进入创建的存储桶
   - 点击"上传文件"
   - 选择 `snake_game.html` 文件上传
   - 上传完成后，点击文件名查看详情
   - 复制"对象地址"即为游戏访问链接

5. **配置自定义域名（可选）**
   - 在存储桶的"域名与传输管理" > "自定义 CDN 加速域名"
   - 添加您的域名
   - 按照提示完成域名验证和CNAME配置

6. **配置静态网站（可选但推荐）**
   - 在存储桶中点击"基础配置" > "静态网站"
   - 开启静态网站功能
   - 索引文档：填写 `snake_game.html`
   - 保存配置
   - 使用提供的静态网站访问地址访问游戏

#### 费用说明：
- COS有免费额度（每月50GB存储，10GB流量）
- 超出部分按量付费，费用很低
- 详情：https://cloud.tencent.com/document/product/436/6239

---

### 方案二：使用腾讯云服务器（CVM）

适合需要更多控制权或者有其他后端服务的场景。

#### 步骤：

1. **购买/登录云服务器**
   - 访问 https://console.cloud.tencent.com/cvm
   - 如果没有服务器，购买一台（推荐：1核2G，系统选Ubuntu 20.04或CentOS 7）

2. **连接服务器**
   ```bash
   ssh root@your_server_ip
   # 输入密码登录
   ```

3. **安装Nginx（推荐）或Apache**
   
   **Ubuntu/Debian系统：**
   ```bash
   sudo apt update
   sudo apt install nginx -y
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```
   
   **CentOS系统：**
   ```bash
   sudo yum install nginx -y
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

4. **上传游戏文件**
   
   **方法1：使用SCP从本地上传**
   ```bash
   # 在本地电脑执行
   scp snake_game.html root@your_server_ip:/usr/share/nginx/html/
   ```
   
   **方法2：直接在服务器创建文件**
   ```bash
   # 在服务器上执行
   cd /usr/share/nginx/html/
   # 使用vim或nano编辑器创建文件，复制粘贴内容
   sudo vim snake_game.html
   # 或者使用wget从GitHub下载
   sudo wget https://raw.githubusercontent.com/cxbrucewayen/cxtest/copilot/create-snake-game-html-file/snake_game.html
   ```

5. **配置防火墙**
   ```bash
   # 开放80端口（HTTP）
   sudo firewall-cmd --permanent --add-service=http
   sudo firewall-cmd --reload
   
   # 或者使用iptables
   sudo iptables -I INPUT -p tcp --dport 80 -j ACCEPT
   ```

6. **在腾讯云安全组配置**
   - 进入云服务器控制台
   - 选择您的服务器 > 安全组
   - 添加入站规则：协议TCP，端口80，来源0.0.0.0/0

7. **访问游戏**
   - 浏览器打开：`http://your_server_ip/snake_game.html`

#### 费用说明：
- 1核2G服务器：约50-100元/月
- 新用户可能有优惠活动

---

### 方案三：使用腾讯云Serverless（云函数 SCF + API网关）

适合按需使用，流量少时费用极低。

#### 步骤：

1. **登录腾讯云控制台**
   - 访问 https://console.cloud.tencent.com/scf

2. **创建云函数**
   - 点击"新建"
   - 函数类型：选择"从头开始"
   - 函数名称：snake-game
   - 运行环境：选择 Python3.x 或 Node.js
   - 函数代码：使用以下代码

   **Python版本：**
   ```python
   # -*- coding: utf8 -*-
   import json
   
   HTML_CONTENT = '''
   [将snake_game.html的全部内容粘贴在这里]
   '''
   
   def main_handler(event, context):
       return {
           "statusCode": 200,
           "headers": {
               "Content-Type": "text/html; charset=utf-8"
           },
           "body": HTML_CONTENT
       }
   ```

3. **配置API网关触发器**
   - 在函数配置页面，点击"触发管理" > "创建触发器"
   - 触发方式：选择"API网关触发器"
   - 按照提示创建API网关
   - 创建完成后会获得访问URL

4. **访问游戏**
   - 使用API网关提供的URL访问游戏

#### 费用说明：
- 每月有免费额度（100万次调用，40万GBs资源使用）
- 超出按量付费，流量少时几乎免费

---

## 🔧 部署前准备

### 环境要求
- 无特殊要求，任何支持HTML5的现代浏览器即可运行
- 支持的浏览器：Chrome、Firefox、Safari、Edge等

### 文件完整性检查
确保 `snake_game.html` 文件完整，大小约为16KB。

---

## 🌐 域名配置（可选）

如果您希望使用自定义域名：

1. **购买域名**
   - 在腾讯云或其他域名注册商购买域名

2. **域名备案**
   - 中国大陆服务器需要完成ICP备案
   - 访问：https://console.cloud.tencent.com/beian

3. **配置DNS解析**
   - 添加A记录指向您的服务器IP
   - 或添加CNAME记录指向COS/CDN域名

---

## 📱 功能特性

- ✅ 完全离线可用（单HTML文件）
- ✅ 响应式设计，支持PC和移动端
- ✅ 支持键盘控制（方向键/WASD）
- ✅ 支持触摸控制（移动设备滑动）
- ✅ 自动保存最高分（使用localStorage）
- ✅ 游戏速度随分数递增
- ✅ 精美的渐变配色和动画效果

---

## 🔍 故障排查

### 问题1：无法访问网站
- 检查服务器防火墙和安全组配置
- 确认Nginx服务正在运行：`systemctl status nginx`
- 检查文件路径是否正确

### 问题2：游戏无法正常运行
- 确保使用现代浏览器
- 按F12查看浏览器控制台是否有错误
- 清除浏览器缓存后重试

### 问题3：移动端无法操作
- 确保使用触摸滑动操作
- 检查浏览器是否支持触摸事件

---

## 💡 性能优化建议

1. **使用CDN加速**
   - 腾讯云COS可以直接开启CDN
   - 提升全国各地访问速度

2. **开启Gzip压缩**
   - Nginx配置：
   ```nginx
   gzip on;
   gzip_types text/html text/css application/javascript;
   ```

3. **配置浏览器缓存**
   - Nginx配置：
   ```nginx
   location ~* \.(html|css|js)$ {
       expires 7d;
   }
   ```

---

## 📞 技术支持

- 腾讯云官方文档：https://cloud.tencent.com/document
- 腾讯云工单系统：https://console.cloud.tencent.com/workorder
- 对象存储文档：https://cloud.tencent.com/document/product/436
- 云服务器文档：https://cloud.tencent.com/document/product/213

---

## 📄 许可说明

本项目代码可自由使用、修改和分发。

---

**祝您部署顺利！玩得开心！🎮**
