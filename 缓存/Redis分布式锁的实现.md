## 方案1
```php
class Lock
{
    private $redis='';  #存储redis对象

    public function __construct($host,$port=6379)
    {
        $this->redis=new Redis();
        $this->redis->connect($host,$port);
    } 

    /**
    * @desc 加锁方法
    *
    * @param $lockName string | 锁的名字
    * @param $timeout int | 锁的过期时间
    *
    * @return 成功返回identifier/失败返回false
    */
    public function getLock($lockName, $timeout=2)
    {
        $identifier=uniqid();       #获取唯一标识符
        $timeout=ceil($timeout);    #确保是整数
        $end=time()+$timeout;
        while(time()<$end)          #循环获取锁
        {
            if($this->redis->setnx($lockName, $identifier))    #查看$lockName是否被上锁
            {
                $this->redis->expire($lockName, $timeout);     #为$lockName设置过期时间，防止死锁
                return $identifier;                             #返回一维标识符
            } elseif ($this->redis->ttl($lockName)===-1) {　　　　　　　　　　　　　　　　　　　　　　　　　　　　　 　
                $this->redis->expire($lockName, $timeout);     #检测是否有设置过期时间，没有则加上（假设，客户端A上一步没能设置时间就进程奔溃了，客户端B就可检测出来，并设置时间）
            }
            usleep(0.001);         #停止0.001ms
        }
        return false;
    }

    /**
    * @desc 释放锁
    *
    * @param $lockName string   | 锁名
    * @param $identifier string | 锁的唯一值
    *
    * @param bool
    */
    public function releaseLock($lockName,$identifier)
    {
        if($this->redis->get($lockName)==$identifier)   #判断是锁有没有被其他客户端修改
        { 
            $this->redis->multi();
            $this->redis->del($lockName);   #释放锁
            $this->redis->exec();
            return true;
        }
        else
        {
            return false;   #其他客户端修改了锁，不能删除别人的锁
        }
    }

    /**
    * @desc 测试
    * 
    * @param $lockName string | 锁名
    */
    public function test($lockName)
    {
        $start=time();
        for ($i=0; $i < 10000; $i++) 
        { 
            $identifier=$this->getLock($lockName);
            if($identifier)
            {
              $count=$this->redis->get('count');
              $count=$count+1;
              $this->redis->set('count',$count);
              $this->releaseLock($lockName,$identifier);
            } 
        }
        $end=time();
        echo "this OK<br/>";
        echo "执行时间为：".($end-$start);
    }
}
```
## 方案2
```php
class RedisMutexLock
{
    /**
     * 缓存 Redis 连接。
     *
     * @return void
     */
    public static function getRedis()
    {
        // 这行代码请根据自己项目替换为自己的获取 Redis 连接。
        return YCache::getRedisClient();
    }

    /**
     * 获得锁,如果锁被占用,阻塞,直到获得锁或者超时。
     * -- 1、如果 $timeout 参数为 0,则立即返回锁。
     * -- 2、建议 timeout 设置为 0,避免 redis 因为阻塞导致性能下降。请根据实际需求进行设置。
     *
     * @param  string  $key         缓存KEY。
     * @param  int     $timeout     取锁超时时间。单位(秒)。等于0,如果当前锁被占用,则立即返回失败。如果大于0,则反复尝试获取锁直到达到该超时时间。
     * @param  int     $lockSecond  锁定时间。单位(秒)。
     * @param  int     $sleep       取锁间隔时间。单位(微秒)。当锁为占用状态时。每隔多久尝试去取锁。默认 0.1 秒一次取锁。
     * @return bool 成功:true、失败:false
     */
    public static function lock($key, $timeout = 0, $lockSecond = 20, $sleep = 100000)
    {
        if (strlen($key) === 0) {
            // 请更换为自己项目抛异常的方法。
            YCore::exception(500, '缓存KEY没有设置');
        }
        $start = self::getMicroTime();
        $redis = self::getRedis();
        do {
            // [1] 锁的 KEY 不存在时设置其值并把过期时间设置为指定的时间。锁的值并不重要。重要的是利用 Redis 的特性。
            $acquired = $redis->set("Lock:{$key}", 1, ['NX', 'EX' => $lockSecond]);
            if ($acquired) {
                break;
            }
            if ($timeout === 0) {
                break;
            }
            usleep($sleep);
        } while (!is_numeric($timeout) || (self::getMicroTime()) < ($start + ($timeout * 1000000)));
        return $acquired ? true : false;
    }

    /**
     * 释放锁
     *
     * @param  mixed  $key  被加锁的KEY。
     * @return void
     */
    public static function release($key)
    {
        if (strlen($key) === 0) {
            // 请更换为自己项目抛异常的方法。
            YCore::exception(500, '缓存KEY没有设置');
        }
        $redis = self::getRedis();
        $redis->del("Lock:{$key}");
    }

    /**
     * 获取当前微秒。
     *
     * @return bigint
     */
    protected static function getMicroTime()
    {
        return bcmul(microtime(true), 1000000);
    }
}
```