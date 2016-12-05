# add HTTPS for sfwn.me

> Date:  2018-06-19
>
> Author: sfwn

```bash
# acme.sh --renew-all
# cp -f \*.sfwn.me.cer \*.sfwn.me.key /etc/nginx/ssl/sfwn.me_and_\*.sfwn.me/
# systemctl reload nginx
```

如果 --renew-all 出问题了，尝试重签。

```bash
# acme.sh --issue --dns dns_ali -d sfwn.me --force
# acme.sh --issue --dns dns_ali -d *.sfwn.me --force
```