![Work of art](./img/snowman_house_of_horror_01.webp)
## Deploy Frontend to EC2

Build Deployable static htmlks using `npm run build` , then use scp to copy to EC2

```
scp -r -i "/Users/vikrantsingh/Desktop/AWS/StoryWeaver/EC2/MyWebServerKey.pem" build  ec2-user@ec2-3-82-154-50.compute-1.amazonaws.com:
```


## Deploy BACKEND to EC2

Build Deployable unit run `sbt assembly`, then use scp to copy to EC2

```
scp -i "/Users/vikrantsingh/Desktop/AWS/StoryWeaver/EC2/MyWebServerKey.pem" /Users/vikrantsingh/homepro-backend-scala/homepro-backend-scala/target/scala-3.5.0/http-assembly-0.1.0.jar ec2-user@ec2-3-82-154-50.compute-1.amazonaws.com:
```

# Installing Java on EC2 

```
sudo dnf update
sudo dnf install java-11-amazon-corretto
```
# Install nginx
```
sudo yum install nginx -y
```

# Serving API from Nginx

Add following block to nginx

```
       location /api/ {
            proxy_pass http://localhost:2101;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
```

# Starting Backend Server

Create a service file - `/etc/systemd/system/homepro-backend.service` 

should look like this 

```
[ec2-user@ip-172-31-68-207 ~]$ cat /etc/systemd/system/homepro-backend.service
[Unit]
Description=Backend for Home Pro
After=network.target

[Service]
ExecStart=java -jar /home/ec2-user/http-assembly-0.1.0.jar
WorkingDirectory=/home/ec2-user
Restart=always
RestartSec=10
User=ec2-user
Environment=JAVA_OPTS="-Xms256m -Xmx512m"

[Install]
WantedBy=multi-user.target
```

Manage using following commands

```
sudo systemctl daemon-reload

cat /etc/systemd/system/homepro-backend.service
sudo systemctl start homepro-backend
sudo systemctl enable homepro-backend // < --- to have it start by default
sudo systemctl status homepro-backend
```
# check Backend logs

```
tail -f logs/app.json
```

# run backend without service

```
sudo java -jar http-assembly-0.1.0.jar
```

# serving React App from Nginx

Nginx need to have following block
```
	location / {
	    root /home/ec2-user/build;
	    index index.html;
            try_files $uri /index.html;
    }
```    

build folder shoud be having nginx user

```
sudo chmod -R 755 /home/ec2-user/build
sudo chown -R nginx:nginx /home/ec2-user/build
```
Check using 

```
drwxr-xr-x.  3 nginx    nginx       16384 Dec 31 18:21 build
```
# Nginx start/stop nginx 

```
sudo systemctl stop nginx
sudo systemctl start nginx
```

# Nginx  Tail access/error logs

```
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

# Nginx  Validate config

```
sudo nginx -t
```

# Nginx Config file

```
/etc/nginx/nginx.conf
```

# Reload Nginx

```
sudo systemctl reload nginx
```

# SSl cert location on Nginx

```
/etc/letsencrypt/live/home-owners.tech/fullchain.pem
```
# SSL cert status on Ngingx

```
sudo certbot certificates
```

# Install certbot
```
sudo yum install certbot -y
```

# SSL CERT Dependency
Files on following folders are needed. When renewing certificates, Let’s Encrypt needs to verify domain ownership.
It places a temporary challenge file in the .well-known/acme-challenge/ directory.

```
/usr/share/nginx/html/.well-known/acme-challenge/
```


