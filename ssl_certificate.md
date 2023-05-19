# How to extract certificates from downloaded bundle

1. Extract the SSL/TLS certificate and private key from the .p7b file. You can do this using the following command:
```
openssl pkcs7 -print_certs -in /etc/ssl/risitibox_com.p7b -out /etc/ssl/risitibox_com.crt

```

2. Create a file using a text editor and paste your private key and rename the file to example.key. Then run this command to create .pem file

```
openssl rsa -in example.key -out example_ssl_key.pem

```

3. Run this command until you get
```
sudo nginx -t
```
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
# SSL
## 1. Install Certbot

https://certbot.eff.org/

For last resort refer this link to use Certbot certificates instead

```
https://realpython.com/django-nginx-gunicorn/
```
https://realpython.com/django-nginx-gunicorn/
