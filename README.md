# Server-HTTP2-CURL-XAMPP-Ubuntu
    This is the memo practice when I go to implement the new APNS HTTP2 API and found my server not support and try to do some alter-way.
    Document detail it run in the Ubuntu 18.04, XAMPP5 or XAMPP7.

# Env to check the CURL whether it support HTTPS or not

## Check OS CURL and XAMPP CURL
    $ curl -V
      curl 7.47.0 (x86_64-pc-linux-gnu) libcurl/7.47.0 GnuTLS/3.4.10 zlib/1.2.8 libidn/1.32 librtmp/2.3
      Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtmp rtsp smb smbs smtp smtps telnet tftp 
      Features: AsynchDNS IDN IPv6 Largefile GSS-API Kerberos SPNEGO NTLM NTLM_WB SSL libz TLS-SRP UnixSockets 

    $ /opt/lampp/bin/curl -V
      curl 7.45.0 (x86_64-pc-linux-gnu) libcurl/7.45.0 OpenSSL/1.0.2j zlib/1.2.8
      Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp smb smbs smtp smtps telnet tftp 
      Features: IPv6 Largefile NTLM NTLM_WB SSL libz TLS-SRP UnixSockets

## We will see there is no HTTP2 in the "Features", it means if we call APNS HTTP2 API below:

    $ curl https://api.push.apple.com:443/3/device/ -k
      curl: (56) GnuTLS recv error (-110): The TLS connection was non-properly terminated.
      @@4???Unexpected HTTP/1.x request: GET /3/device/

    $ /opt/lampp/bin/curl https://api.push.apple.com:443/3/device/ -k
      @@4???Unexpected HTTP/1.x request: GET /3/device/ 

## Install HTTP2 Library and Upgrade CRUL

### Uninstall OS CURL 
    $ apt remove curl && apt purge curl
    
### Install OS CURL and HTTP2 Library
    $ sudo apt-get update -y
    $ sudo apt-get install -y build-essential nghttp2 libnghttp2-dev libssl-dev
    $ wget https://curl.haxx.se/download/curl-7.63.0.tar.gz
    $ sudo tar -xzvf curl-7.63.0.tar.gz && rm -f curl-7.63.0.tar.gz && cd curl-7.63.0
    $ sudo ./configure --prefix=/usr/local --with-ssl --with-nghttp2
    $ sudo make
    $ sudo make install
    $ sudo ldconfig
    $ cd ~ && rm -rf curl-7.63.0
    $ ln -s /usr/local/bin/curl /usr/bin/curl
    $ curl -V 
      curl 7.63.0 (x86_64-pc-linux-gnu) libcurl/7.63.0 OpenSSL/1.0.2g zlib/1.2.8 nghttp2/1.7.1
      Release-Date: 2018-12-12
      Protocols: dict file ftp ftps gopher http https imap imaps pop3 pop3s rtsp smb smbs smtp smtps telnet tftp 
      Features: AsynchDNS IPv6 Largefile NTLM NTLM_WB SSL libz TLS-SRP HTTP2 UnixSockets HTTPS-proxy 

### Change XAMPP CURL to OS one
    $ sudo cp /opt/lampp/bin/curl /opt/lampp/bin/curl.bkup
    $ sudo cp /usr/bin/curl /opt/lampp/bin/curl
    $ /opt/lampp/bin/curl -V
      curl 7.63.0 (x86_64-pc-linux-gnu) libcurl/7.63.0 OpenSSL/1.0.2g zlib/1.2.8 nghttp2/1.7.1
      Release-Date: 2018-12-12
      Protocols: dict file ftp ftps gopher http https imap imaps pop3 pop3s rtsp smb smbs smtp smtps telnet tftp 
      Features: AsynchDNS IPv6 Largefile NTLM NTLM_WB SSL libz TLS-SRP HTTP2 UnixSockets HTTPS-proxy 

### We may see some error occurs when runing program due to "curl: /usr/local/lib/libcurl.so.4: no version information available (required by curl) "
    $ locate libcurl.so.4
      /opt/lampp/lib/libcurl.so.4
      /opt/lampp/lib/libcurl.so.4.4.0
      /usr/lib/x86_64-linux-gnu/libcurl.so.4
      /usr/lib/x86_64-linux-gnu/libcurl.so.4.5.0
      /usr/local/lib/libcurl.so.4
      /usr/local/lib/libcurl.so.4.6.0
    $ ls -l /opt/lampp/lib/libcurl.so.4
      lrwxrwxrwx 1 root root 16 May 31  2017 /opt/lampp/lib/libcurl.so.4 -> libcurl.so.4.4.0
    $ sudo cp /opt/lampp/lib/libcurl.so.4 /opt/lampp/lib/libcurl.so.4.bk
    $ sudo rm /opt/lampp/lib/libcurl.so.4
    $ ln -s /usr/lib/x86_64-linux-gnu/libcurl.so.4 /opt/lampp/lib/libcurl.so.4 
    $ sudo /opt/lampp/lampp restart
    
[Reference](https://stackoverflow.com/a/38896967)

### Try to Call then no HTTP error now
    $ curl https://api.push.apple.com:443/3/device/ -k
    {"reason":"MethodNotAllowed"}
    $ /opt/lampp/bin/curl https://api.push.apple.com:443/3/device/ -k
    {"reason":"MethodNotAllowed"}
    
## Call APNS API then will be received the PUSH
    $ curl -vs  \
        --header "apns-topic: com.xxx.app"  
        --header "apns-push-type: alert"  
        --cert "/absolute_path/apns_cert.pem:your_encrypted_pem_password"  \
        --data '{"aps":{"alert":"test","badge":1}}'  \
        --http2 https://api.push.apple.com:443/3/device/4F15267EE72CAA67DBDC3F6DE2173544E2F16A4FA3521C4EB3F3DBCA974AD331 \
        -k
      
        *   Trying 17.188.163.12...
        * TCP_NODELAY set
        * Connected to api.push.apple.com (17.188.163.12) port 443 (#0)
        * ALPN, offering h2
        * ALPN, offering http/1.1
        * Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
        * successfully set certificate verify locations:
        *   CAfile: /etc/ssl/certs/ca-certificates.crt
          CApath: none
        * TLSv1.2 (OUT), TLS header, Certificate Status (22):
        * TLSv1.2 (OUT), TLS handshake, Client hello (1):
        * TLSv1.2 (IN), TLS handshake, Server hello (2):
        * TLSv1.2 (IN), TLS handshake, Certificate (11):
        * TLSv1.2 (IN), TLS handshake, Server key exchange (12):
        * TLSv1.2 (IN), TLS handshake, Request CERT (13):
        * TLSv1.2 (IN), TLS handshake, Server finished (14):
        * TLSv1.2 (OUT), TLS handshake, Certificate (11):
        * TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
        * TLSv1.2 (OUT), TLS handshake, CERT verify (15):
        * TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
        * TLSv1.2 (OUT), TLS handshake, Finished (20):
        * TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
        * TLSv1.2 (IN), TLS handshake, Finished (20):
        * SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
        * ALPN, server accepted to use h2
        * Server certificate:
        *  subject: CN=api.push.apple.com; OU=management:idms.group.533599; O=Apple Inc.; ST=California; C=US
        *  start date: Mar 14 17:50:10 2019 GMT
        *  expire date: Apr 12 17:50:10 2021 GMT
        *  issuer: CN=Apple IST CA 2 - G1; OU=Certification Authority; O=Apple Inc.; C=US
        *  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
        * Using HTTP2, server supports multi-use
        * Connection state changed (HTTP/2 confirmed)
        * Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
        * Using Stream ID: 1 (easy handle 0xb48070)
        > POST /3/device/4F15267EE72CAA67DBDC3F6DE2173544E2F16Axxxxxxxxxxx HTTP/2
        > Host: api.push.apple.com
        > User-Agent: curl/7.63.0
        > Accept: */*
        > apns-topic: com.xxx.app
        > apns-push-type: alert
        > Content-Length: 34
        > Content-Type: application/x-www-form-urlencoded
        > 
        * Connection state changed (MAX_CONCURRENT_STREAMS == 1000)!
        * We are completely uploaded and fine
        < HTTP/2 200 
        < apns-id: xxxxxx-1C94-4457-092A-746C6B32368B
        < 
        * Connection #0 to host api.push.apple.com left intact
