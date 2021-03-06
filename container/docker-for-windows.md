# Docker for Windows

{% embed url="https://www.docker.com/products/docker-desktop" %}

## 簡介

DevOps議題中難免會提到Serverless、MicroService\(微服務\)，主要原因是希望可以簡單、加速服務的部署並同時減少資源的浪費

Container基本架構如下：OS核心為共用，Container中只要安裝服務所需要的軟體即可\(例如iis\)

![](../.gitbook/assets/image%20%28215%29%20%281%29.png)

傳統VM，每一個網站Server都需要有自己的os核心\(例如windows\)，再外加網站所需的服務\(例如iis\)

然後os核心，以windows為例非常佔空間、記憶體，因此用此方式想做到MicroService勢必需要花費很多資料，故Docker才會在近年被愈來愈多軟體業廣範使用

![Container vs VM](../.gitbook/assets/image%20%28248%29%20%283%29.png)

簡單的說，使用Docker的好處：可以透過建立好的Image檔，快速將服務啟動在Container\(容器\)中

而這台Server本身不需要安裝iis、appach等軟體\(因為這些在Image中已設定完畢\)

以.NET Core網站為例，Images大小約100mb，因此在任意有安裝Docker的Server，透過此Image檔都可以簡單、快速把網站服務建立完畢

## 安裝前準備1：Windows功能開啟 - Containers / Hyper-V

Windows中使用Docker，需要開啟Containers及Hyper-V功能

![](../.gitbook/assets/image%20%28132%29%20%281%29.png)

勾選Containers、Hyper-V

![](../.gitbook/assets/image%20%28214%29%20%281%29.png)

## 安裝前準備2：BIOS - VTx設定

BIOS - Virtualization Technology設定需打開，否則可能在Docker Container Run時出現Container無法運行錯誤

![](../.gitbook/assets/image%20%2831%29.png)

## Docker for Windows安裝

下載路徑：[https://docs.docker.com/docker-for-windows/install/](https://docs.docker.com/docker-for-windows/install/)

![](../.gitbook/assets/image%20%28142%29%20%281%29.png)

安裝大約900mb

![](../.gitbook/assets/image%20%28167%29.png)

安裝過程原則上依指示下一步到底即可 \(此頁面可以把Use Windows Containers選項打勾\)

![](../.gitbook/assets/image%20%28181%29.png)

跳出此警告時，按ok即可\(會重開機\)



![](../.gitbook/assets/image%20%28113%29%20%282%29.png)

重開機完畢後，Docker Desktop會自動開啟如下

![](../.gitbook/assets/image%20%28110%29.png)

開啟命令提示字元\(cmd\)，輸入docker version，若有看到版本號正確顯示即代表docker正常進行中

![](../.gitbook/assets/image%20%2870%29%20%281%29.png)

## Demo - ASP.Net MVC部署在容器

### 1.準備一個建立完成的ASP.Net MVC網站如下

![](../.gitbook/assets/image%20%28105%29%20%281%29.png)

### 2.準備/下載 Asp.Net MVC執行環境的Image檔 \(可從Docker Hub下載\)

ASP.Net MVC\(非.Net Core\)可以選擇ASP.Net 4.8的Image

網址：[https://hub.docker.com/\_/microsoft-dotnet-framework-aspnet/](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/)

docker下載Image可以透過網站中提供指令\(docker pull mcr.microsoft.com/dotnet/framework/aspnet:4.8\)

![](../.gitbook/assets/image%20%28104%29.png)

在cmd中輸入docker pull mcr.microsoft.com/dotnet/framework/aspnet:4.8

\(docker pull 下載Image\)

![](../.gitbook/assets/image%20%2848%29.png)

開始下載Image檔

![](../.gitbook/assets/image%20%28100%29%20%281%29.png)

下載完成後，docker Images可列出所有本機Images

![](../.gitbook/assets/image%20%2867%29%20%281%29.png)

簡易測試Image

-d：detach\(背景執行，docker run後不會綁住原本的命令提示視窗，而是回傳建立好的容器id\)

-p：port mapping \(Host主機的port與容器中port對應設定，否則外面連不進去容器\)

```text
docker run --name testapp1 -p 1000:80 -d 7becd5ccb930
```

![](../.gitbook/assets/image%20%2852%29.png)

執行後，可以輸入docker ps確認目前運行中的容器

![](../.gitbook/assets/image%20%2818%29%20%281%29.png)

確認容器\(container\)運作無誤後，開啟browser，連線至localhost:1000

測試iis服務

出現以下錯誤，即代表iis運作正常，但無內容

![](../.gitbook/assets/image%20%28130%29.png)

進入container - testapp1，並開啟cmd

```text
docker exec -it testapp1 cmd
```

![](../.gitbook/assets/image%20%28144%29.png)

![](../.gitbook/assets/image%20%2855%29%20%281%29.png)

進入inetpub/wwwroot，確認無檔案

![](../.gitbook/assets/image%20%2838%29%20%281%29.png)

產生測試用頁面

```text
echo test > index.aspx
```

![](../.gitbook/assets/image%20%28210%29.png)

測試

![](../.gitbook/assets/image%20%2823%29%20%281%29.png)

### 3.將檔案放置進Container中

asptestweb目錄如下

![](../.gitbook/assets/image%20%28197%29%20%282%29.png)

![](../.gitbook/assets/image%20%28193%29%20%282%29.png)

docker容器中可以mount位置到Host磁碟 \(讓容器可以取得host資料\)，因此先刪除已建立的容器testapp1

重建testapp1，並加入-v指令 去指定host中的d:\webfiles\asptestweb對應到容器中c:\webfile資料夾

```text
docker stop testapp1
docker rm testapp1
```

```text
docker run --name testapp1 -p 1000:80 -v D:\WebFile\ASPTestWeb:c:\webfile -d 7becd
```

![](../.gitbook/assets/image%20%2865%29%20%281%29.png)

進入容器testapp1後，可以看到webfile資料\(其會對應host的d:\webfile\asptestweb資料夾\)

```text
docker exec -it testapp1 cmd
```

![](../.gitbook/assets/image%20%28151%29.png)

將檔案copy到inetpub/wwwroot資料夾\(iis根目錄\)

```text
xcopy /s c:\webfile c:\inetpub\wwwroot
```

![](../.gitbook/assets/image%20%28185%29%20%281%29.png)

![](../.gitbook/assets/image%20%2889%29.png)

copy完成後，可以測試網站localhost:1000

![](../.gitbook/assets/image%20%2815%29.png)

### 4.將container封裝成Image檔

如果要建一個裡面已經有特定服務/網頁的Image時，可以使用docker commit指令將目前的container封裝成Image檔 \(一般ci/cd流程不一定會這麼做\)

```text
docker commit container_id image_name
```

commit‌前，需先將container 停止

```text
docker stop testapp1
docker commit testapp1 img_test
```

![](../.gitbook/assets/image%20%28209%29%20%281%29.png)

![](../.gitbook/assets/image%20%2825%29%20%282%29.png)

建立完成後，可以在image列表中查詢到該image

```text
docker images
```

![](../.gitbook/assets/image%20%28166%29%20%283%29.png)

透過新建的img\_test建立一個新的容器測試testapp2

```text
docker run --name testapp2 -p 2000:80 -d img_test
```

![](../.gitbook/assets/image%20%2874%29%20%282%29.png)

容器testapp2建立完成後，即可以測試localhost:2000

![](../.gitbook/assets/image%20%28190%29%20%282%29.png)

透過此方式，即可以快速部署web應用程式至任何有docker的環境上，該server不需要安裝iis、.net framework及相關複雜的設定，因此這些設定都在建立image時已經設定完畢

**小結**：

img\_test是利用asp.net 4.8 Image建立出來的Image，其Size為**5.66GB**

testweb則是使用asp.net core nanoserver版\(精簡\) Image建立的Image，可以發現Size僅有**519MB**，主要原因為asp.net為了支援跨平台，因此不需綁定許多windows元件，因此空間會相對小許多 \(如果需要entity framework或其它功能仍可再加裝上去\)

使用容器主要目的為serverless、microservice，去除不必要的gui介面、不需要的元件後，Image可以做到100mb以下，如此可以讓服務啟動時間大幅增加、減少不要必的磁碟、記憶體使用

\(一台Docker Server可以起N個容器服務\)

![](../.gitbook/assets/image%20%28140%29%20%281%29.png)

### 5.使用Dockfile建立Image

上述1-4動作，可以寫成Dockfile，在Dockfile中編寫執行的動作，執行docker bulid指令後可以簡化Image製作流程、增加彈性

例如以下：

![](../.gitbook/assets/image%20%28177%29%20%282%29.png)

以下說明使用方法：

假設已建立完成網站如下：

![](../.gitbook/assets/image%20%28155%29%20%281%29.png)

在此根目錄新增檔案Dockerfile \(不一定要叫這個名稱\)，內容空白即可，並去除附檔名

![](../.gitbook/assets/image%20%28195%29.png)

用文字編輯器開啟Dockerfile\(建議用vscode\)

![](../.gitbook/assets/image%20%28194%29%20%283%29.png)

Dockerfile大致上需要做到的是

1. 設定基底Image來源：輸入FROM 基底Image名稱或dockhub上下載位置
2. 設定Image預設工作目錄\(預設資料夾\)：輸入WORKDIR 預設的工作目錄位置
3. 將網站檔案Copy至Workdir：輸入COPY 來源\(本機\) 目的地\(Image內\)

**Step1-設定基底Image來源**：因為demo用的網站為asp.net mvc，所以選擇asp.net 4.8的Image

```text
FROM mcr.microsoft.com/dotnet/framework/aspnet:4.8
```

**Step2-設定Image預設工作目錄**：希望預設工作目錄在IIS目錄下\(C:\INETPUB\WWWROOT\)

```text
WORKDIR C:/INETPUB/WWWROOT
```

**Step3-將網站檔案Copy至Workdir：**因為dockerfile的位置已經放在asp網站檔案根目錄下，所以可以用「.」指定當前目錄\(來源\)，至於Image內目的地資料夾，亦可以用「.」來表示，因為預設工作目錄已經在c:\inetpub\wwwroot下

```text
COPY . .
```

編寫完的Dockerfile檔如下：

![](../.gitbook/assets/image%20%28128%29.png)

在網站目錄下，執行cmd進到命令提示字元視窗

快速的方式為用檔案總管到該資料下

![](../.gitbook/assets/image%20%2863%29.png)

將上方的位置列改成cmd

![](../.gitbook/assets/image%20%28109%29%20%281%29.png)

按下enter即可

![](../.gitbook/assets/image%20%2881%29%20%281%29.png)

接下用docker指令去建置docker image檔並使用dockerfile進行相關設定

指令如下：

```text
docker build -f Dockerfile -t testweb:1.0 .
```

docker build為建置image指令

-f 為指定dockerfile檔案名稱，因此輸入-f Dockerfile

-t 為給定建立的image tag名稱及版本號

最後一個「.」意指在docker build時，需要使用哪些檔案，因為希望把當前目錄資料全部放置進image中，所以可以直接輸入「.」

在cmd中執行上述指令

![](../.gitbook/assets/image%20%28146%29.png)

![](../.gitbook/assets/image%20%2822%29%20%283%29.png)

執行成功後，可以輸入docker images檢示目前所有Image

![](../.gitbook/assets/image%20%28157%29.png)

測試建立好的Image檔，使用docker run去產生container\(容器\)

容器名稱：testapp3

Port：3000

Image：testweb

```text
docker run --name testapp3 -p 3000:80 -d testweb
```

成功Run起容器

![](../.gitbook/assets/image%20%2886%29.png)

測試網站 http://localhost:3000

![](../.gitbook/assets/image%20%2859%29%20%281%29.png)

利用此法即可以透過指令化方式，建立一個Image並包含web應用程式服務，因此若有另一台server需要建立此服務，即可以直接下載該Image後，就可以直接快速部署一個網站

一般container結合ci/cd流程時，通常也都會搭配使用dockerfile方式，去進行專案建置、nuget還原、copy檔案等作業，如下圖，相關指令可以參與docker官網介紹或是使用Visual Studio工具自動產生

![](../.gitbook/assets/image%20%28177%29%20%282%29.png)

```csharp
FROM microsoft/dotnet:2.1-aspnetcore-runtime-nanoserver-1803 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM microsoft/dotnet:2.1-sdk-nanoserver-1803 AS build
WORKDIR /src
COPY ["TestWeb/TestWeb.csproj", "TestWeb/"]
RUN dotnet restore "TestWeb/TestWeb.csproj"
COPY . .
WORKDIR "/src/TestWeb"
RUN dotnet build "TestWeb.csproj" -c Release -o /app

FROM build AS publish
RUN dotnet publish "TestWeb.csproj" -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "TestWeb.dll"]
```

### 6.Visual Studio整合Docker

Visual Studio 2019以後，有跟Docker整合，可以直接在Visual Studio中快速產生Docker Image並且可以在Container中進行debug

假設現有的網站專案如下

![](../.gitbook/assets/image%20%28129%29%20%281%29.png)

用Visual Studio 2019開啟，在專案按右鍵→加入→容器支援 \(英文版Docker Support\)

![](../.gitbook/assets/image%20%28120%29%20%281%29.png)

點選後，Visual Studio將會針對專案進行Docker相關設定及檢查

![](../.gitbook/assets/image%20%2857%29%20%282%29.png)

若本地沒有需要的image時，可能需要花一點時間進行image下載

![](../.gitbook/assets/image%20%2884%29.png)

完成後，專案內會產生Dockerfile

![](../.gitbook/assets/image%20%2826%29.png)

專案偵測目標可以選擇docker

![](../.gitbook/assets/image%20%2897%29%20%281%29.png)

Debug模式下 - 可以在container內偵錯，如下圖，其中172.26.197.81為container的虛擬ip

![](../.gitbook/assets/image%20%28203%29.png)

可以觀察按下建置後，可以發現Visual Studio其實是先建置後，再執行Docker Build、docker run指令

```text
docker build -f "D:\vs2017\ASPWebTest\ASPWebTest\Dockerfile" --force-rm -t aspwebtest  --label "com.microsoft.created-by=visual-studio" --label "com.microsoft.visual-studio.project-name=ASPWebTest" "D:\vs2017\ASPWebTest\ASPWebTest"
```

![](../.gitbook/assets/image%20%28148%29.png)

因此在cmd中輸入docker images，可以看到Image會被產生

release模式：tag為lastest；debug模式：tag為dev

![](../.gitbook/assets/image%20%28159%29%20%281%29.png)

輸入docker ps，可以看到容器被建立

![](../.gitbook/assets/image%20%28178%29%20%282%29.png)

因此輸入localhost:63377 \(每次可能不一樣\)，亦可進入到網站

![](../.gitbook/assets/image%20%28172%29.png)

使用Visual Studio Docker整合功能，可以更簡單地使用Docker

一般在偵錯時，建議還是先選擇IIS Express \(不用做docker build / docker run\)

![](../.gitbook/assets/image%20%28202%29%20%281%29.png)

