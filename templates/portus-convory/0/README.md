Server端安裝流程：
1. 產生自簽名憑證檔案
    portos 與 registry 需要憑證用以支援加密協定。
    以下流程將介紹如何產生自簽名憑證並在環境中安裝認證檔案
        
    a. 建立認證檔案與私有鑰匙
        $ openssl req -x509 -nodes -day 3650 -newkey rsa:2048 -keyout server-key.pem -out server-crt.pem
    
    c. 在Rancher/Storage 建立volume。 volume 名稱為： ${PREFIX}-certs 
        將認證檔 server-key.pem 與 server-crt.pem 放入 ${PREFIX}-certs
        Hint:
            ${PREFIX} 為一段字串，需與 [Convory volume prefix] 欄位設定相同


2. Host Label 標籤
    portus 需要將容器都建立在要分享的機器上
    在Rancher/Host/Edit 中增加標籤，registry_host=true。
    Hint:
        此設定需與 [Host Label] 欄位設定相同

3. 建立第一個帳號
    如果尚未建立過帳號，登入頁面將會直接提示建立帳號訊息。(此帳號將會有管理者權限)
    之後登入頁面以提示登入畫面為主

4. 建立 registry information
    以第一個帳號登入後，將會直接引導至registry建立頁面，範例如下：
    name:   your-name
    hostname: your-FQDN:registry-port
    use ssl: YES
    
5. Portus operate account
    登入 portus web後，需要建立背景處理使用的帳號 portus
    (若尚未建立此帳號則Admin 頁面將會提示 The Portus user does not exist!)。
    進入Admin/user 頁面建立帳號
    name: portus
    email: anyone@email.any.com
    password: your-password
    Hint:
        此帳號的密碼需與 [Portus Operate Password] 設定相同

Client端安裝流程：
1. 建立 ssl key 目錄位置
    $ mkdir -p /etc/docker/certs.d/<your-FQDN>/

2. 安裝憑證
    確認系統中已安裝  ca-certificates 套件

    ubuntu：
    $ cp ca.crt /usr/local/share/ca-certificates/<your-FQDN>.crt
    $ update-ca-certificates

    Red Hat:
    $ cp ca.crt /etc/pki/ca-trust/source/anchors/<your-FQDN>.crt
    $ update-ca-trust

    Hint:
    On some distributions, e.g. Oracle Linux 6, the Shared System Certificates feature needs to be manually enabled:
    $ update-ca-trust enable
    reference: https://docs.docker.com/registry/insecure/

    $ cp ca.crt /etc/docker/certs.d/<your-FQDN:registry-port>/ca.crt
    $ service docker restart
    
3. 登入 docker registry
    $ docker login -u user -p password -e email@mail.com <your-FQDN:registry-port>
    