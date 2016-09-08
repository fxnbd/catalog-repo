1. 憑證檔案
    portos 與 registry 需要憑證用以支援加密協定。
    以下流程將介紹如何產生自簽名憑證並在環境中安裝認證檔案
    a. 編輯ssl.conf
        以下幾個欄位需要填寫
        $ vim ssl.conf
       
        [ req ]
        prompt             = no
        distinguished_name = req_subj
        x509_extensions    = x509_ext
        
        [ req_subj ]
        CN = YOUR-FQDN
        
        [ x509_ext ]
        subjectKeyIdentifier   = hash
        authorityKeyIdentifier = keyid,issuer
        basicConstraints       = CA:true
        subjectAltName         = @alternate_names
        
        [ alternate_names ]
        DNS.1 = YOUR-FQDN
        
        DNS.2 = YOUR-HOSTNAME
        IP.2  = YOUR-CURRENT-IP-ADDRESS
        .
        .
        .
    
    b. 建立認證檔案與私有鑰匙
        openssl req -config ssl.conf -new -x509 -nodes \
        -sha256 -days 3650 -newkey rsa:4096 -keyout server-key.pem -out server-crt.pem
    
    c. 在Rancher/Storage 建立volume。 volume 名稱為： ${PREFIX}-certs 
        將認證檔 server-key.pem 與 server-crt.pem 放入 ${PREFIX}-certs
        ${PREFIX} 為一段字串，需與 [Convory volume prefix] 欄位設定相同


2. Host Label 標籤
    portus 需要將容器都建立在要分享的機器上
    在Rancher/Host/Edit 中增加標籤，registry_host=true。
    此設定需與 [Host Label] 欄位設定相同

3. 建立第一個帳號
    如果尚未建立過帳號，登入頁面將會直接提示建立帳號訊息。(此帳號將會有管理者權限)
    之後登入頁面以提示登入畫面為主

4. 建立 registry information
    以第一個帳號登入後，將會直接引導至registry建立頁面，範例如下：
    name:   your-name
    hostname: your-FQDN:registry-port
    use ssl: YES
    
3. Portus operate account
    登入 portus web後，需要建立背景處理使用的帳號 portus
    (若尚未建立此帳號則Admin 頁面將會提示 The Portus user does not exist!)。
    進入Admin/user 頁面建立帳號，此帳號的密碼需與 [Portus Operate Password] 設定相同
    name: portus
    email: anyone@email.any.com
    password: your-password

4. Client 連線
    將認證檔放入client端此位置
    /etc/docker/cert.d/<your-FQDN:registry-port>/ca.crt
    重啟 docker engine後 即可登入
    docker login -u user -p password -e email@mail.com <your-FQDN:registry-port>
    