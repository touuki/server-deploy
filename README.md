# server-deploy
服务部署记录

## 通配符域名申请
```bash
certbot-auto certonly  -d *.example.com --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```
刷新`certbot-auto renew`
