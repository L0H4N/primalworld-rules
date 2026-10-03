# เว็บกฎแบบแยกเครื่อง

เว็บนี้เป็น Static Website ไฟล์ที่ต้องนำขึ้น Server อยู่ในโฟลเดอร์ `public`

## วิธีแนะนำ: `rules.primalworld.online`

วิธีนี้แยกเว็บกฎออกจากเว็บหลักและแยก Server ได้สมบูรณ์

### 1. ตั้ง DNS

สร้าง Record ในผู้ให้บริการ DNS:

```text
Type: A
Name: rules
Value: IP สาธารณะของ Server ใหม่
```

### 2. นำไฟล์ขึ้น Server

```bash
sudo mkdir -p /var/www/primal-world-rules
sudo cp -r public/* /var/www/primal-world-rules/
```

### 3. ตั้ง Nginx

นำ `nginx-rules-subdomain.conf` ไปไว้ที่:

```text
/etc/nginx/sites-available/primal-world-rules
```

แล้วเปิดใช้งาน:

```bash
sudo ln -s /etc/nginx/sites-available/primal-world-rules /etc/nginx/sites-enabled/primal-world-rules
sudo nginx -t
sudo systemctl reload nginx
```

### 4. เปิด HTTPS

```bash
sudo certbot --nginx -d rules.primalworld.online
```

หน้าเว็บจะเปิดที่ `https://rules.primalworld.online/`

## ถ้าต้องใช้ `primalworld.online/rulepage/`

DNS ไม่สามารถส่งเฉพาะ `/rulepage` ไปอีก Server ได้ เครื่องที่ให้บริการ `primalworld.online` ต้องเพิ่ม Reverse Proxy ดังนี้:

```nginx
location = /rulepage {
    return 301 /rulepage/;
}

location /rulepage/ {
    proxy_pass http://IP_SERVER_ใหม่:8080/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

บน Server ใหม่ให้ใช้ `nginx-rulepage-backend.conf` และนำไฟล์ใน `public` ไปไว้ที่ `/var/www/primal-world-rules`

ควรจำกัด Firewall ของ Port 8080 ให้รับการเชื่อมต่อจาก IP ของ Server เว็บหลักเท่านั้น
