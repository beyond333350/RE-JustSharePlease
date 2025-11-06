# RE-JustSharePlease
This is the revised version of ObsidianJustSharePlease
原作者地址：  https://github.com/Ellpeck/ObsidianJustSharePlease 
这是一个非常棒且简洁的Obsidian分享的项目，在该作者基础上修改了以下几处：
1. 分享页面的自适应宽度
2. 让浏览器title自动识别分享也的标题
3. 将分享页面底部信息移除，影响美观
4. 修改分享面失效提示

---

## 环境与目录规划

- 操作系统：Ubuntu 20.04 / 22.04 / 24.04
- 域名：`your-domain.com`（已在 DNS 指向服务器）
- 服务路径：`/opt/jsp/server/public`
- 后端监听：`127.0.0.1:8888`（仅本机监听，Nginx 反代暴露 HTTPS）

```bash
sudo mkdir -p /opt/jsp/server/public
sudo chown -R $USER:$USER /opt/jsp
```

安装 PHP 与 Nginx
方案 A：PHP 内置开发服务器（快速部署）

```
sudo apt update
sudo apt install -y php php-cli php-curl php-xml php-mbstring unzip
php -v
```

将 JSP 服务端文件放入 /opt/jsp/server/public，至少包含：
share.php
index.html
index.js
style.css
可选：favicon.ico、index.md

创建数据目录与权限：
```
mkdir -p /opt/jsp/server/public/data
chmod 775 /opt/jsp/server/public/data
```

#方案 B：Nginx + PHP-FPM（推荐生产）
```
sudo apt update
sudo apt install -y nginx php-fpm php-curl php-xml php-mbstring unzip
php -v
systemctl status php8.1-fpm || systemctl status php8.2-fpm
```

systemd 开机自启（针对方案 A：php -S）

```
sudo tee /etc/systemd/system/jsp.service >/dev/null <<'EOF'
[Unit]
Description=Just Share Please PHP Dev Server
After=network.target

[Service]
User=www-data
WorkingDirectory=/opt/jsp/server/public
ExecStart=/usr/bin/php -S 127.0.0.1:8888 -t /opt/jsp/server/public
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable jsp.service
sudo systemctl start jsp.service
systemctl status jsp.service
```

Nginx 反向代理与 HTTPS（Let’s Encrypt）
安装与签发证书：
```
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

配置文件：

```
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate     /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options SAMEORIGIN;
    add_header Referrer-Policy strict-origin-when-cross-origin;

    location / {
        proxy_pass http://127.0.0.1:8888;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

```

启用与重载：

```
sudo ln -s /etc/nginx/sites-available/jsp.conf /etc/nginx/sites-enabled/jsp.conf
sudo nginx -t
sudo systemctl reload nginx
```
Obsidian 插件配置与前端定制
插件 Server URL（必须 HTTPS）

```
https://your-domain.com
```
