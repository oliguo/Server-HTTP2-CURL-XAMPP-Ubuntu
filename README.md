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

  We will see there is no HTTP2 in the "Features", it means if we call APNS HTTP2 API below:
  
  $ curl https://api.push.apple.com:443/3/device/ -k
    curl: (56) GnuTLS recv error (-110): The TLS connection was non-properly terminated.
    @@4???Unexpected HTTP/1.x request: GET /3/device/
  
  $ /opt/lampp/bin/curl https://api.push.apple.com:443/3/device/ -k
    @@4???Unexpected HTTP/1.x request: GET /3/device/ 
  
  
  
