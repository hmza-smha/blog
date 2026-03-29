
[Back](./README.md)

---

## 📑 Table of Contents

| Section | Topic |
|---------|-------|
| [1](#-what-is-the-full-chain) | What is Full Chain? |
| [2](#-why-do-you-need-it) | Why Full Chain? |
| [3](#-update-nginx-config) | Update NGINX Config |
| [4](#-step-6-test--reload) | Test & Reload |
| [5](#-what-i-did-in-the-new-nginx-config) | NGINX Config Explained |

---

# Adding SSL Certificate To Ubuntu and NGINX

`cd /etc/nginx/`

`mkdir ssl`

> Move SSL files to /etc/nginx/ssl/

Build the full chain

🔹 What is the "full chain"?

When your browser connects to a website over HTTPS, it doesn’t just check the server certificate (e.g. `wildcard.xyz.com.cer`). It also needs to verify the trust chain all the way up to a root Certificate Authority (CA) that it already trusts.

- Your domain/server certificate proves ownership of `xyz.com`.

- The intermediate certificates (in `chain.crt`) “bridge the gap” between your certificate and the root.

- The root certificate is already stored in browsers/OS — that’s why you don’t need to provide it yourself.

When you concatenate your certificate with the intermediates (`fullchain.pem`), you’re essentially packaging everything the browser needs to validate your site’s identity.

🔹 Why do you need it?

If you configure NGINX with just your certificate:

- Some browsers may fail to trust your site.

- You’ll get SSL errors like “certificate not trusted” or “incomplete chain”.

- Tools like SSL Labs will show a broken trust chain.

By using the fullchain, you’re ensuring that all browsers and clients can validate the certificate consistently.

👉 In short:

`wildcard.xyz.com.cer` = your site’s certificate

`chain.crt` = the intermediate link(s)

`fullchain.pem` = both of the above combined

That’s why we build the `fullchain.pem` with:

`cat wildcard.xyz.com.cer chain.crt > fullchain.pem`


🔹Update NGINX config


```
server {
    listen 80;
    listen [::]:80;
    server_name xyz.com *.xyz.com;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name xyz.com *.xyz.com;

    ssl_certificate     /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privatekey.pem;

    client_max_body_size 500M;

    proxy_read_timeout 3600;
    proxy_connect_timeout 3600;
    proxy_send_timeout 3600;

    location / {
        allow 94.249.61.230;
        allow 49.13.102.175;
        allow 78.46.117.251;
        deny all;

        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /dev/ {
        allow 94.249.61.230;
        allow 49.13.102.175;
        allow 78.46.117.251;
        deny all;

        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

🔹 Step 6: Test & reload

Check syntax:
```
sudo nginx -t
```

If OK, reload:
```
sudo systemctl reload nginx
```

🔹 What I did in the new NGINX config
1. Added a redirect server block (port 80 → 443)
```
server {
    listen 80;
    listen [::]:80;
    server_name xyz.com *.xyz.com;

    return 301 https://$host$request_uri;
}
```

- Listens on port 80 (HTTP).

- Any request coming to `http://xyz.com` or `http://*.xyz.com` is redirected to HTTPS (`301` permanent redirect).

- This ensures your users don’t stay on insecure HTTP.


2. Created the HTTPS server block
```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name xyz.com *.xyz.com;
```

- Listens on port ```443 (HTTPS)``` with `ssl` enabled.
- Uses http2 for faster connections.
- Matches both your base domain (`xyz.com`) and any subdomains (`*.xyz.com`), since you’re using a wildcard certificate.


### The reason https://49.13.102.175/ is still available is because NGINX is falling back to a default SSL server block that listens on 443 for all requests (when no server_name matches).

We can explicitly disable serving HTTPS on the raw IP by doing one of these:

🔹 Option 1: Add a “catch-all” block for the IP and drop requests
server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;
    server_name _;

    ssl_certificate     /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privatekey.pem;

    return 444;  # close connection without response
}


default_server ensures this block catches anything that doesn’t match your domain.

server_name _; is a wildcard placeholder (matches everything).

return 444; tells NGINX to just close the connection (client gets no response).

Now only aiserver.esensesoftware.com will serve content, not the IP.

🔹 Option 2: Redirect IP → Domain

If you’d prefer to redirect users hitting the IP to your domain:

server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;
    server_name _;

    ssl_certificate     /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privatekey.pem;

    return 301 https://aiserver.esensesoftware.com$request_uri;
}


This way if someone visits https://49.13.102.175/, they’ll land on https://aiserver.esensesoftware.com/.

🔹 What I recommend

- If you want to completely block IP access, go with Option 1 (clean, secure).

- If you don’t mind people reaching the domain via IP and just want them redirected, go with Option 2.

---
[Back](./README.md)