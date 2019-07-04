在使用分布式对数据进行存储时，经常会碰到需要新增节点来满足业务快速增长的需求。然而在新增节点时，如果处理不善会导致所有的数据重新分片，这对于某些系统来说可能是灾难性的。
那么是否有可行的方法，在数据重分片时，只需要迁移与之关联的节点而不需要迁移整个数据呢？当然有，在这种情况下我们可以使用一致性Hash来处理。
![原理图](https://image.slidesharecdn.com/cassandra-basic-161014090027/95/cassandra-basic-8-638.jpg?cb=1476437202)

# 说明
- 用一定的哈希算法（哈希函数等）将一组服务器的多个（数目自己设定）节点随机映射分散到0-2的32次方之间，由于其随机分布，保证了其数据平均分布的特点；
- 用同一算法计算要存储数据的键，根据服务器节点确定其存储的服务器结点，由于每次用同一算法计算，所以得出的结果是相同的，使其查找定位准确；
- 查找数据时，再次用同一算法计算键，并查找服务器的数据结点；
- 从数据映射到的位置开始顺时针查找，将数据保存到找到的第一个服务器上。如果超过2的32次方仍然找不到服务器，就会保存到第一台memcached服务器上。
- 如果有一个服务器宕机，消除其服务器结点，并将数据放在下一个结点上，由于随机节点位置的随机性，所以数据被其他服务器平均负载，也就降低了宕机影响。

# PHP实现

```php
// 一致性哈希算法
class ConsistentHashing
{
    protected $nodes = array();    //真实节点
    protected $position = array();  //虚拟节点
    protected $mul = 64;  // 每个节点对应64个虚拟节点

    /**
     * 把字符串转为32位符号整数
     */
    public function hash($str)
    {
        return sprintf('%u', crc32($str));
    }

    /**
     * 核心功能
     */
    public function lookup($key)
    {
        $point = $this->hash($key);

        //先取圆环上最小的一个节点,当成结果
        $node = current($this->position);

        // 循环获取相近的节点
        foreach ($this->position as $key => $val) {
            if ($point <= $key) {
                $node = $val;
                break;
            }
        }

        reset($this->position);    //把数组的内部指针指向第一个元素，便于下次查询从头查找

        return $node;
    }

    /**
     * 添加节点
     */
    public function addNode($node)
    {
        if(isset($this->nodes[$node])) return;

        // 添加节点和虚拟节点
        for ($i = 0; $i < $this->mul; $i++) {
            $pos = $this->hash($node . '-' . $i);
            $this->position[$pos] = $node;
            $this->nodes[$node][] = $pos;
        }

        // 重新排序
        $this->sortPos();
    }

    /**
     * 删除节点
     */
    public function delNode($node)
    {
        if (!isset($this->nodes[$node])) return;

        // 循环删除虚拟节点
        foreach ($this->nodes[$node] as $val) {
            unset($this->position[$val]);
        }

        // 删除节点
        unset($this->nodes[$node]);
    }

    /**
     * 排序
     */
    public function sortPos()
    {
        ksort($this->position, SORT_REGULAR);
    }
}

// 测试
$con = new ConsistentHashing();

$con->addNode('a');
$con->addNode('b');
$con->addNode('c');
$con->addNode('d');

$key1 = 'www.zhihu.com';
$key2 = 'www.baidu.com';
$key3 = 'www.google.com';
$key4 = 'www.testabc.com';

echo 'key' . $key1 . '落在' . $con->lookup($key1) . '号节点上！<br>';
echo 'key' . $key2 . '落在' . $con->lookup($key2) . '号节点上！<br>';
echo 'key' . $key3 . '落在' . $con->lookup($key3) . '号节点上！<br>';
echo 'key' . $key4 . '落在' . $con->lookup($key4) . '号节点上！<br>';
```