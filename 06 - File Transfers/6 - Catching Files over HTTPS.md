We can use **Nginx** as an alternative for Apache.

**Create a Directory to Handle Uploaded Files:**

```sh
sudo mkdir -p /var/www/uploads/SecretUploadDirectory
```

**Change the Owner to www-data:**

```sh
sudo chown -R www-data:www-data /var/www/uploads/SecretUploadDirectory
```

**Create Nginx Configuration File:**

Create the Nginx configuration file by creating the file `/etc/nginx/sites-available/upload.conf` with the contents:

```sh
server {
    listen 9001;
    
    location /SecretUploadDirectory/ {
        root    /var/www/uploads;
        dav_methods PUT;
    }
}
```

**Symlink our Site to the sites-enabled Directory:**

```sh
sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/
```

**Start Nginx:**

```sh
sudo systemctl restart nginx.service
```

**Uploading Files using cURL:**

```sh
curl -T /etc/passwd http://localhost:9001/SecretUploadDirectory/users.txt
```

