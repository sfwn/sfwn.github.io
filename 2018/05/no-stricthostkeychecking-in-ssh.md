# no-stricthostkeychecking-in-ssh

> Date:  2018-05-24
>
> Author: sfwn
>
> Title: ssh: StrictHostKeyChecking=no

~/.ssh/config

```bash
Host *
    StrictHostKeyChecking no
    UserKnownHostsFile=/dev/null
```

https://askubuntu.com/questions/87449/how-to-disable-strict-host-key-checking-in-ssh