## OVERVIEW

The command dig is a tool for querying DNS nameservers for information about host addresses, mail exchanges, nameservers, and related information. This tool can be used from any Linux (Unix) or Macintosh OS X operating system. The most typical use of dig is to simply query a single host.

## INSTRUCTIONS

### Run the command:

```
 dig mt-example.com
```

### View the Output:

```
; <<>> DiG 9.4.1-P1 <<>> mt-example.com
;; global options:  printcmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25550
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;mt-example.com.			IN	A

;; ANSWER SECTION:
mt-example.com.		28626	IN	A	205.186.150.66

;; Query time: 4 msec
;; SERVER: 64.207.129.21#53(64.207.129.21)
;; WHEN: Thu Aug  7 16:49:35 2008
;; MSG SIZE  rcvd: 48
```

### Understanding the Results

The opening section of digâ€™s output tells us a little about itself (version 9.4.1) and the global options that are set (in this case, `printcmd`):

```
; <<>> DiG 9.4.1-P1 <<>> mt-example.com
;; global options:  printcmd
```

Here, dig tells us some technical details about the answer received from the DNS server. This section of the output can be toggled using the +[no]comments optionâ€”but beware that disabling the comments also turns off many section headers:

```
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25550
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
```

In the question section, dig reminds us of our query. The default query is for an Internet address (A).

```
;; QUESTION SECTION:
;mt-example.com.			IN	A
```

Now we have our anwser, the address of mt-example.com is 205.186.150.66.

```
;; ANSWER SECTION:
mt-example.com.		28626	IN	A	205.186.150.66
```

The final section of the default output contains statistics about the query; it can be toggled with the +[no]statsoption.

```
;; Query time: 272 msec
;; SERVER: 208.67.222.222#53(208.67.222.222)
;; WHEN: Thu Feb 13 09:35:55 PST 2014
;; MSG SIZE  rcvd: 48
```

A quick way to just get the answer only is to run the following command:

```
dig mt-example.com +short
```

### What can I find using the dig command?

**dig** will let you perform any valid DNS query, the most common of which are:

- A (the IP address),
- TXT (text annotations),
- MX (mail exchanges), and
- NS nameservers.

Use the following command to get the addresses for mt-example.com.

```
 dig mt-example.com A +noall +answer
```

Use the following command to get a list of all the mailservers for mt-example.com.

```
 dig mt-example.com MX +noall +answer
```

Use the following command to get a list of authoritative DNS servers for mt-example.com.

```
 dig mt-example.com NS +noall +answer
```

Use the following command to get a list of all the above in one set of results.

```
dig mt-example.com ANY +noall +answer 
```

Use the following command to query using a specific nameserver.

```
dig @ns1.mediatemple.net mt-example.com 
```

Use the following to trace the path taken.

```
 dig mt-example.com +trace
```



#### 遇到过的问题

* dig 返回的 DNS 记录不全

dig any 获取到的记录不全，具体原因待查

