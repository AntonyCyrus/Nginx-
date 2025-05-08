Nginx反向代理模板

```bash
# 可选 HTTP 配置，强制跳转 HTTPS
server {
    listen 80;
    server_name your-vps-domain.com;
    return 301 https://$host$request_uri;
}
server {
    listen 443 ssl http2;
    server_name your-vps-domain.com;

    # 使用自签名证书或 Let's Encrypt
    ssl_certificate /etc/ssl/certs/your_cert.pem;             # SSL 证书路径
    ssl_certificate_key /etc/ssl/private/your_key.pem;        # SSL 密钥路径

    # 强制使用安全协议和加密套件
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # 日志路径（可根据需要开启或关闭）
    access_log /var/log/nginx/proxy_access.log;
    error_log /var/log/nginx/proxy_error.log;

    # 设置最大请求体（可选）
    client_max_body_size 10M;

    # 目标网站的反向代理路径（如 https://target.com）
    location / {
        proxy_pass https://target.com;

        # 取消默认添加的代理头，避免泄露真实 IP
        proxy_set_header Host target.com;              # 保持目标网站 Host 不变
        proxy_set_header X-Real-IP "";                 # 移除客户端真实 IP
        proxy_set_header X-Forwarded-For "";           # 移除代理链信息
        proxy_set_header Via "";                       # 移除 Via 头
        proxy_hide_header X-Powered-By;                # 隐藏服务器标识

        # 模拟真实浏览器的 User-Agent（可选）
        proxy_set_header User-Agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64)";

        # 禁止缓存，防止信息泄露
        proxy_set_header Cache-Control "no-cache";
        proxy_set_header Pragma "no-cache";

        # 设置连接超时
        proxy_connect_timeout 10s;
        proxy_read_timeout 30s;

        # 禁用 WebSocket 升级（可选，根据目标网站情况）
        proxy_set_header Upgrade "";
        proxy_set_header Connection "close";

        # 启用 TLS 验证（确保使用 HTTPS 回源）
        proxy_ssl_verify on;
        proxy_ssl_trusted_certificate /etc/ssl/certs/ca-certificates.crt;
        proxy_ssl_server_name on;
    }

    # 强制跳转 HTTP 到 HTTPS（可选）
    error_page 497 https://$host$request_uri;
}
```
