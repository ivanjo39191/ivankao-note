# Request wildcard Certificate with acme.sh in Docker
Let's Encrypt Free Certificate

Created: February 1, 2022 5:22 PM
Tags: OS

- validity 90 days
- wildcard Yes
- multiple main domains Yes

```docker
# step 1
docker run --rm  -it  \
  -v "/opt/cert/wildcard":/acme.sh  \
  neilpang/acme.sh --issue \
  -d "*.ivankaoblog.com" \
  --register-account -m ivankao82500@gmail.com \
  --dns --yes-I-know-dns-manual-mode-enough-go-ahead-please

# step 2: update TXT record for DNS verification

# step 3
docker run --rm  -it  \
  -v "/opt/cert/wildcard":/acme.sh  \
  neilpang/acme.sh --renew \
  -d "*.ivankaoblog.com" \
  --dns --yes-I-know-dns-manual-mode-enough-go-ahead-please
```

[https://www.feitsui.com/en/article/19](https://www.feitsui.com/en/article/19)