# OpenSSL SMIME加密

## 通过rsa算法生成私钥

```bash
#采用RSA算法
$ openssl genrsa -out rsakey0.pem 1024
```

## 通过私钥文件生成证书

```bash
#产生RSA算法的证书
$ openssl req -x509 -key rsakey0.pem -days 365 -out mycert-rsa.pem -new
```

## 使用证书加密文件

```bash
openssl smime -sign -signer mycert-rsa.pem -inkey rsakey0.pem -md sha1 -text -in message -out lisence2.txt
```