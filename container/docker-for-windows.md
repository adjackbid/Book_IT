# Docker for Windows

{% embed url="https://www.docker.com/products/docker-desktop" %}



![](../.gitbook/assets/image%20%28172%29.png)



## 安裝前準備1：Windows功能開啟 - Containers / Hyper-V

Windows中使用Docker，需要開啟Containers及Hyper-V功能

![](../.gitbook/assets/image%20%28110%29.png)

勾選Containers、Hyper-V

![](../.gitbook/assets/image%20%28171%29.png)

## 安裝前準備2：BIOS - VTx設定

BIOS - Virtualization Technology設定需打開，否則可能在Docker Container Run時出現Container無法運行錯誤

![](../.gitbook/assets/image%20%2825%29.png)

## Docker for Windows安裝

下載路徑：[https://docs.docker.com/docker-for-windows/install/](https://docs.docker.com/docker-for-windows/install/)

![](../.gitbook/assets/image%20%28119%29.png)

安裝大約900mb

![](../.gitbook/assets/image%20%28136%29.png)

安裝過程原則上依指示下一步到底即可 \(此頁面可以把Use Windows Containers選項打勾\)

![](../.gitbook/assets/image%20%28146%29.png)

跳出此警告時，按ok即可\(會重開機\)



![](../.gitbook/assets/image%20%2895%29.png)

重開機完畢後，Docker Desktop會自動開啟如下

![](../.gitbook/assets/image%20%2892%29.png)

開啟命令提示字元\(cmd\)，輸入docker version，若有看到版本號正確顯示即代表docker正常進行中

![](../.gitbook/assets/image%20%2859%29.png)

## Demo - ASP.Net MVC部署在容器

### 1.準備一個建立完成的ASP.Net MVC網站如下

![](../.gitbook/assets/image%20%2888%29.png)

### 2.準備/下載 Asp.Net MVC執行環境的Image檔 \(可從Docker Hub下載\)

ASP.Net MVC\(非.Net Core\)可以選擇ASP.Net 4.8的Image

網址：[https://hub.docker.com/\_/microsoft-dotnet-framework-aspnet/](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/)

docker下載Image可以透過網站中提供指令\(docker pull mcr.microsoft.com/dotnet/framework/aspnet:4.8\)

![](../.gitbook/assets/image%20%2887%29.png)

在cmd中輸入docker pull mcr.microsoft.com/dotnet/framework/aspnet:4.8

\(docker pull 下載Image\)

![](../.gitbook/assets/image%20%2841%29.png)

開始下載Image檔

![](../.gitbook/assets/image%20%2883%29.png)

下載完成後，docker Images可列出所有本機Images

![](../.gitbook/assets/image%20%2856%29.png)

簡易測試Image

-d：detach

-p：port mapping \(Host主機的port與容器中port對應設定\)

```text
docker run --name testapp1 -p 1000:80 -d 7becd5ccb930
```

![](../.gitbook/assets/image%20%2844%29.png)

執行後，可以輸入docker ps確認目前運行中的容器

![](../.gitbook/assets/image%20%2815%29.png)

確認容器\(container\)運作無誤後，開啟browser，連線至localhost:1000

測試iis服務

出現以下錯誤，即代表iis運作正常，但無內容

![](../.gitbook/assets/image%20%28108%29.png)

進入container - testapp1，並開啟cmd

```text
docker exec -it testapp1 cmd
```

![](../.gitbook/assets/image%20%28121%29.png)

![](../.gitbook/assets/image%20%2847%29.png)

進入inetpub/wwwroot，確認無檔案

![](../.gitbook/assets/image%20%2832%29.png)

產生測試用頁面

```text
echo test > index.aspx
```

![](../.gitbook/assets/image%20%28167%29.png)

測試

![](../.gitbook/assets/image%20%2819%29.png)

### 3.將檔案放置進Container中

asptestweb目錄如下

![](../.gitbook/assets/image%20%28157%29.png)

![](../.gitbook/assets/image%20%28155%29.png)

docker容器中可以mount位置到Host磁碟 \(讓容器可以取得host資料\)，因此先刪除已建立的容器testapp1

重建testapp1，並加入-v指令 去指定host中的d:\webfiles\asptestweb對應到容器中c:\webfile資料夾

```text
docker stop testapp1
docker rm testapp1
```

```text
docker run --name testapp1 -p 1000:80 -v D:\WebFile\ASPTestWeb:c:\webfile -d 7becd
```

![](../.gitbook/assets/image%20%2854%29.png)

進入容器testapp1後，可以看到webfile資料\(其會對應host的d:\webfile\asptestweb資料夾\)

```text
docker exec -it testapp1 cmd
```

![](../.gitbook/assets/image%20%28126%29.png)

將檔案copy到inetpub/wwwroot資料夾\(iis根目錄\)

```text
xcopy /s c:\webfile c:\inetpub\wwwroot
```

![](../.gitbook/assets/image%20%28150%29.png)

![](../.gitbook/assets/image%20%2874%29.png)

copy完成後，可以測試網站localhost:1000

![](../.gitbook/assets/image%20%2812%29.png)

### 4.將container封裝成Image檔

























