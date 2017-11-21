### certification file parse

#### generate certification file
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=yourcompany.com" -days 5000 -out ca.crt

#### parse certification file
openssl x509 -in certificate.crt -text -noout

```
Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number: 14024472537904156962 (0xc2a0ef77e7ff9122)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=yourcompany.com
        Validity
            Not Before: Nov 21 01:50:34 2017 GMT
            Not After : Jul 31 01:50:34 2031 GMT
        Subject: CN=yourcompany.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:b8:52:43:a5:b2:85:c7:a0:d4:2a:55:0b:d8:88:
                    8a:05
                Exponent: 65537 (0x10001)
    Signature Algorithm: sha256WithRSAEncryption
         70:bc:f9:85:75:63:8b:41:42:da:21:73:36:41:87:7e:3a:21:
         2c:a5:04:e0
```

