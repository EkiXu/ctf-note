# 一些有用的指令
```
tcpdump -n -r nmapll.pcapng 'tcp[13] = 18' | awk '{print $3}'| sort -u
```