# CopyFailDemo
## Test if the `algif_aead` module loaded 
```
lsmod | grep -q "^algif_aead" && echo "[VULN] LOADED" || echo "[INFO] not loaded"
```

## Check the CRYPTO_USER_API config
```
if zcat /proc/config.gz 2>/dev/null | grep -q "^CONFIG_CRYPTO_USER_API=y"; then
    echo "[VULN] BUILT-IN"; V=1
elif zcat /proc/config.gz 2>/dev/null | grep -q "^CONFIG_CRYPTO_USER_API=m"; then
    echo "[INFO] MODULE (trying to load...)"
    modprobe algif_aead 2>/dev/null
    lsmod | grep -q "^algif_aead" && echo "[VULN] loaded OK"
else
    echo "[SAFE] not enabled"
fi
```

## Check authencesn algorithm
```
grep -q "authencesn" /proc/crypto 2>/dev/null && echo "[VULN] available" || echo "[SAFE] not found"
```

## Check AF_ALG soceket
```
python3 -c "
import socket
try:
    s=socket.socket(38,5,0)
    s.bind(('aead','authencesn(hmac(sha256),cbc(aes))'))
    print('[VULN] socket created OK')
    exit(0)
except Exception as e:
    print('[SAFE] failed:',e)
    exit(1)
" 2>/dev/null

```
