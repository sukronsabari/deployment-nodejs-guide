# deployment-nodejs-guide

## Langkah 1: Update Sistem dan Konfigurasi Swap Memory
### Update Sistem:
```
sudo apt update
```

### Konfigurasi Swap Memory (opsional, jika RAM kecil):
```
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo swapon --show
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

Install NVM untuk Mengelola Node.js dan npm
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.nvm/nvm.sh
nvm install --lts
```

### Clone Repository Project dan Build Aplikasi
Clone Repository dari GitHub:
```
git clone https://github.com/user/repository.git
cd repository
```

### Build Aplikasi (untuk Next.js jika build error):
```
node --max-old-space-size=2048 node_modules/.bin/next build
```

### Install dan Konfigurasi PM2
```
npm install pm2 -g
```

Jadikan PM2 Running pada Startup:
```
pm2 startup
```

Tambahkan Aplikasi ke PM2:
```
pm2 start npm --name "nama-aplikasi" -- script

pm2 save
```

### Konfigurasi Nginx sebagai Reverse Proxy
Install Nginx:
```
sudo apt install nginx -y
```

Aktifkan Firewall dan Izinkan SSH serta HTTP/HTTPS:
```
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

Hapus Konfigurasi Default Nginx:
```
sudo rm /etc/nginx/sites-available/default
sudo rm /etc/nginx/sites-enabled/default
```

Buat Konfigurasi Nginx Baru untuk Setiap Domain:
```
sudo nano /etc/nginx/sites-available/namaconfig
```

Isi dengan konfigurasi berikut:
Setiap server satu saja
```
server {
    listen 80;
    server_name casecobra.cloud;

    location / {
        proxy_pass http://localhost:9000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

server {
    listen 80;
    server_name casecobra1.cloud;

    location / {
        proxy_pass http://localhost:9001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

server {
    listen 80;
    server_name casecobra2.cloud;

    location / {
        proxy_pass http://localhost:9002;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

```

Aktifkan Konfigurasi Nginx Baru:
```
sudo ln -s /etc/nginx/sites-available/namaconfig /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### Mengamankan Website dengan SSL Menggunakan Certbot
Install Certbot dan Plugin Nginx:
```
sudo apt install certbot python3-certbot-nginx -y
```

Dapatkan dan Install Sertifikat SSL:
```
sudo certbot --nginx -d 'namadomain.cloud'
```

Restart Nginx:
```
sudo systemctl restart nginx
```
