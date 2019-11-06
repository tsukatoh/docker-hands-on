# Docker ハンズオン

## **1. Dockerコマンドの確認**
* docker pull
* dokcer images
* docker run
* docker ps
* docker exec
* docker stop
* docker rm
* docker rmi

### **docker pull**
コンテナーイメージをレジストリからダウンロードするコマンドです。  
実際にコマンドを実行してnginxのコンテナーイメージをダウンロードしてみます。  

```
docker pull nginx
```

以下のように表示されれば完了です。  

```
Using default tag: latest
latest: Pulling from library/nginx
8d691f585fa8: Already exists
5b07f4e08ad0: Pull complete
abc291867bca: Pull complete
Digest: sha256:922c815aa4df050d4df476e92daed4231f466acc8ee90e0e774951b0fd7195a4
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```


### **docker images**
ダウンロード済みのコンテナーイメージを確認するコマンドです。  
先ほどダウンロードしたコンテナーイメージを確認してみます。  

```
docker images
```

ダウンロードされたイメージが表示されます。  

```
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
nginx                                      latest              540a289bab6c        7 days ago          126MB
```


### **docker run**
コンテナーイメージを実行するコマンドです。  
ポートのフォワードや終了後に実行イメージを削除するなどのオプションがあります。  
先ほどのnginxのイメージを実行してみましょう。  

```
docker run -it --name nginx -p 8080:80 nginx
```


### **docker ps**
実行中のコンテナーを表示するコマンドです。  
先ほど起動したコンテナーを確認してみます。  

```
docker ps
```


### **docker exec**
実行中のコンテナーでコマンドを実行するコマンドです。  
bashを指定してshellを起動してみます。  

```
docker exec -it nginx bash
```

以下コマンドで動作しているOSの情報を確認してみます。  

```
cat /etc/os-release
```


### **docker stop**
実行中のコンテナーを停止するコマンドです。  
起動中のコンテナーを停止します。  

```
docker stop nginx
```

docker ps コマンドで確認してみます。  

```
docker ps
```

実行中コンテナーがなくなりました。  
停止中コンテナーを確認するには -a オプションを使います。  

```
docker ps -a
```


### **docker rm**
停止したコンテナーを削除するコマンドです。  

```
docker rm nginx
```

削除されたか確認してみます。  

```
docker ps -a
```

なくなっていれば削除完了です。  


### **docker rmi**
Dockerイメージを削除するコマンドです。  
docker images コマンドで確認して対象のイメージを削除します。  

```
docker images
docker rmi nginx
```


## **2. カスタムコンテナの作成**
* docker build
* docker run

自分でDockerfileを作成して、カスタムコンテナーを作る方法です。  

### **docker build**
Dockerfileを元にコンテナーをビルドするコマンドです。  
サンプルをビルドしてみます。  
※サンプルのDockerfileがあるディレクトリで作業をしてください。  

```
docker build -t customnginx .
```

ビルドされたら「customnginx」という名前でコンテナーイメージが作成されているか確認します。  

```
docker images
```

以下のように登録されていれば完成です。  

```
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
customnginx                                latest              12188150cfcb        5 minutes ago       205MB
```

実際に動かしてみましょう。  
コンテナ側のポート80をホスト側のポート8080へマッピングして起動させます。  

```
docker run -it --name customnginx -p 8080:80 customnginx
```

起動できたらWebブラウザでhttp://localhost:8080へアクセスしてみます。  

![localhost](/img/localhost.png)

上記の様に表示されたら完了です。  


## **3. クラウドにデプロイしてみよう**
* Azure Container Registry
* Azure Container Instances
* docker login
* docker tag
* docker push

### **Azure Container Registry**
Azure Container Registry（ACR）はコンテナーイメージを保管するコンテナーレジストリサービスです。  
AzureポータルからCloud Shellを起動してAzure CLIで作成していきます。  
まずリソースグループを作成します。  

```
az group create --name kumaazu --location japanwest
```

レジストリ名は一意の名前をつける必要があります。  
管理者ユーザーは有効に変更し、SKUはBasicを選択します。  

```
az acr create --resource-group kumaazu --name {Registry_Name} --sku Basic --admin-enabled
```

すぐに作成できますので、loginServerを控えておきます。  
アクセスするためのキーを確認します。  

```
az acr credential show --name {Registry_Name}
```

username、passwordを控えておいてください。  

次に作成したコンテナーをコンテナーレジストリにプッシュします。  
作業しているコンソールにて以下コマンドを入力してコンテナーレジストリへログインします。  
認証に使用するのは先ほど控えたユーザー名、passwordです。  

```
docker login {ログインサーバー}
```

ログイン出来たら、作ったコンテナーイメージのタグを変更してプッシュできる状態にします。  

```
docker tag customnginx {ログインサーバー}/customnginx:v1
```

変更できたらACRへプッシュします。  

```
docker push {ログインサーバー}/customnginx:v1
```

ここまででコンテナーレジストリへの登録は完了です。  

### **Azure Container Instances**
Azure Container Instances（ACI）はサーバーを管理することなくAzureでコンテナーを簡単に実行できるマネージドなコンテナーサービスです。  
コマンド1つでコンテナーの実行が可能になります。  

以下コマンドで作成します。  
Cloud Shellにて操作します。  
--dns-name-labelは一意の値にする必要があります。  

```
az container create --resource-group kumaazu --name kumaazu-demo --image {ログインサーバー}/customnginx:v1 --dns-name-label {ACI_DNS_Name} --ports 80
```

作成が完了したらFQDNを確認します。  

```
az container show --resource-group kumaazu --name kumaazu-demo --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" --out table
```

```
FQDN                                  ProvisioningState
------------------------------------  -------------------
kumademo.japanwest.azurecontainer.io  Succeeded
```

実際にFQDNでアクセスしてみましょう。  

先ほどローカルで動かしたWeb画面が表示されれば完了です。  

## **4. wordpressコンテナを作ってみよう**
* docker-compose
* Web App for Containers
