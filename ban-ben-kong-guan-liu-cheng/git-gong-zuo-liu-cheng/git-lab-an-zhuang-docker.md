# Git Lab安裝\(Docker\)

Windows上使用Docker安裝GitLab

將Docker mode切換至Linux

![](../../.gitbook/assets/image%20%283%29.png)

docker hub可以找到官方建立好的gitlab Image

[https://hub.docker.com/r/gitlab/gitlab-ce/](https://hub.docker.com/r/gitlab/gitlab-ce/)

![](../../.gitbook/assets/image%20%2866%29.png)

cmd中輸入

```text
docker pull gitlab/gitlab-ce
```

![](../../.gitbook/assets/image%20%28224%29.png)

建立Container

```text
docker run -d -p 443:443 -p 8080:80 -p 22:22 --name gitlab 
--restart always --hostname gitlab.cim 
-v /gitlab/config:/etc/gitlab:z -v /gitlab/logs:/var/log/gitlab:z 
-v /gitlab/data:/var/opt/gitlab:z gitlab/gitlab-ce:latestw
```

測試：http://localhost:8080

![](../../.gitbook/assets/image%20%28220%29.png)

1到2分鐘後，再點一次icon

輸入密碼

![](../../.gitbook/assets/image%20%28267%29.png)

登入號帳為root

![](../../.gitbook/assets/image%20%28128%29.png)

登入後，即可以新增帳號



![](../../.gitbook/assets/image%20%289%29.png)

ToDo：smtp設定、Demo Project Create...

