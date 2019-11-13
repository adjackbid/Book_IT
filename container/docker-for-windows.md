# Docker for Windows

{% embed url="https://www.docker.com/products/docker-desktop" %}



![](../.gitbook/assets/image%20%28179%29.png)



## 安裝前準備1：Windows功能開啟 - Containers / Hyper-V

Windows中使用Docker，需要開啟Containers及Hyper-V功能

![](../.gitbook/assets/image%20%28112%29.png)

勾選Containers、Hyper-V

![](../.gitbook/assets/image%20%28178%29.png)

## 安裝前準備2：BIOS - VTx設定

BIOS - Virtualization Technology設定需打開，否則可能在Docker Container Run時出現Container無法運行錯誤

![](../.gitbook/assets/image%20%2826%29.png)

## Docker for Windows安裝

下載路徑：[https://docs.docker.com/docker-for-windows/install/](https://docs.docker.com/docker-for-windows/install/)

![](../.gitbook/assets/image%20%28122%29.png)

安裝大約900mb

![](../.gitbook/assets/image%20%28140%29.png)

安裝過程原則上依指示下一步到底即可 \(此頁面可以把Use Windows Containers選項打勾\)

![](../.gitbook/assets/image%20%28151%29.png)

跳出此警告時，按ok即可\(會重開機\)



![](../.gitbook/assets/image%20%2897%29.png)

重開機完畢後，Docker Desktop會自動開啟如下

![](../.gitbook/assets/image%20%2894%29.png)

開啟命令提示字元\(cmd\)，輸入docker version，若有看到版本號正確顯示即代表docker正常進行中

![](../.gitbook/assets/image%20%2860%29.png)

## Demo - ASP.Net MVC部署在容器

### 1.準備一個建立完成的ASP.Net MVC網站如下

![](../.gitbook/assets/image%20%2890%29.png)

### 2.準備/下載 Asp.Net MVC執行環境的Image檔 \(可從Docker Hub下載\)

ASP.Net MVC\(非.Net Core\)可以選擇ASP.Net 4.8的Image

網址：[https://hub.docker.com/\_/microsoft-dotnet-framework-aspnet/](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/)

docker下載Image可以透過網站中提供指令\(docker pull mcr.microsoft.com/dotnet/framework/aspnet:4.8\)

![](../.gitbook/assets/image%20%2889%29.png)

在cmd中輸入docker pull mcr.microsoft.com/dotnet/framework/aspnet:4.8

\(docker pull 下載Image\)

![](../.gitbook/assets/image%20%2842%29.png)

開始下載Image檔

![](../.gitbook/assets/image%20%2885%29.png)

下載完成後，docker Images可列出所有本機Images

![](../.gitbook/assets/image%20%2857%29.png)

簡易測試Image

-d：detach\(背景執行，docker run後不會綁住原本的命令提示視窗，而是回傳建立好的容器id\)

-p：port mapping \(Host主機的port與容器中port對應設定，否則外面連不進去容器\)

```text
docker run --name testapp1 -p 1000:80 -d 7becd5ccb930
```

![](../.gitbook/assets/image%20%2845%29.png)

執行後，可以輸入docker ps確認目前運行中的容器

![](../.gitbook/assets/image%20%2815%29.png)

確認容器\(container\)運作無誤後，開啟browser，連線至localhost:1000

測試iis服務

出現以下錯誤，即代表iis運作正常，但無內容

![](../.gitbook/assets/image%20%28110%29.png)

進入container - testapp1，並開啟cmd

```text
docker exec -it testapp1 cmd
```

![](../.gitbook/assets/image%20%28124%29.png)

![](../.gitbook/assets/image%20%2848%29.png)

進入inetpub/wwwroot，確認無檔案

![](../.gitbook/assets/image%20%2833%29.png)

產生測試用頁面

```text
echo test > index.aspx
```

![](../.gitbook/assets/image%20%28174%29.png)

測試

![](../.gitbook/assets/image%20%2819%29.png)

### 3.將檔案放置進Container中

asptestweb目錄如下

![](../.gitbook/assets/image%20%28163%29.png)

![](../.gitbook/assets/image%20%28161%29.png)

docker容器中可以mount位置到Host磁碟 \(讓容器可以取得host資料\)，因此先刪除已建立的容器testapp1

重建testapp1，並加入-v指令 去指定host中的d:\webfiles\asptestweb對應到容器中c:\webfile資料夾

```text
docker stop testapp1
docker rm testapp1
```

```text
docker run --name testapp1 -p 1000:80 -v D:\WebFile\ASPTestWeb:c:\webfile -d 7becd
```

![](../.gitbook/assets/image%20%2855%29.png)

進入容器testapp1後，可以看到webfile資料\(其會對應host的d:\webfile\asptestweb資料夾\)

```text
docker exec -it testapp1 cmd
```

![](../.gitbook/assets/image%20%28129%29.png)

將檔案copy到inetpub/wwwroot資料夾\(iis根目錄\)

```text
xcopy /s c:\webfile c:\inetpub\wwwroot
```

![](../.gitbook/assets/image%20%28155%29.png)

![](../.gitbook/assets/image%20%2876%29.png)

copy完成後，可以測試網站localhost:1000

![](../.gitbook/assets/image%20%2812%29.png)

### 4.將container封裝成Image檔

因此如果要建一個裡面已經有特定服務/網頁的Image時，可以使用docker commit指令將目前的container封裝成Image檔 \(一般ci/cd流程不一定會這麼做\)

```text
docker commit container_id image_name
```

commit‌前，需先將container 停止

```text
docker stop testapp1
docker commit testapp1 img_test
```

![](../.gitbook/assets/image%20%28173%29.png)

![](../.gitbook/assets/image%20%2821%29.png)

建立完成後，可以在image列表中查詢到該image

```text
docker images
```

![](../.gitbook/assets/image%20%28139%29.png)

透過新建的img\_test建立一個新的容器測試testapp2

```text
docker run --name testapp2 -p 2000:80 -d img_test
```

![](../.gitbook/assets/image%20%2864%29.png)

容器testapp2建立完成後，即可以測試localhost:2000

![](../.gitbook/assets/image%20%28158%29.png)

透過此方式，即可以快速部署web應用程式至任何有docker的環境上，該server不需要安裝iis、.net framework及相關複雜的設定，因此這些設定都在建立image時已經設定完畢

**小結**：

img\_test是利用asp.net 4.8 Image建立出來的Image，其Size為**5.66GB**

testweb則是使用asp.net core nanoserver版\(精簡\) Image建立的Image，可以發現Size僅有**519MB**，主要原因為asp.net為了支援跨平台，因此不需綁定許多windows元件，因此空間會相對小許多 \(如果需要entity framework或其它功能仍可再加裝上去\)

使用容器主要目的為serverless、microservice，去除不必要的gui介面、不需要的元件後，Image可以做到100mb以下，如此可以讓服務啟動時間大幅增加、減少不要必的磁碟、記憶體使用

\(一台Docker Server可以起N個容器服務\)

![](../.gitbook/assets/image%20%28120%29.png)

### 5.使用Dockfile建立Image

上述1-4動作，可以寫成Dockfile，在Dockfile中編寫執行的動作，執行docker bulid指令後可以簡化Image製作流程、增加彈性

例如以下：

![](../.gitbook/assets/image%20%28149%29.png)

To Do：待補Demo實作

### 6.Visual Studio整合Docker

To Do：待補













