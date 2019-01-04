memcached分布式最大的特点就是完全由客户端程序库实现，客户端库根据key通过算法算出应该交互的服务器，然后进行数据的存储与获取，如下图所示：

![](https://github.com/bingbo/blog/blob/master/images/standard_hash.jpg)

## 定位服务器的hash策略

### 标准hash策略(余数计算)

通过CRC32或FNV根据key算出键的hash值，再除以服务器台数，根据余数来选择服务器。

```c
unsigned int hash = mmc_hash(state, key, key_len), i;
mmc = state->buckets[hash % state->num_buckets];
```

_缺点_

当添加或移除服务器时，缓存重组的代价相当巨大。添加服务器后，余数就会产生巨变，这样就无法获取与保存时相同的服务器，从而影响缓存的命中率

## 持久hash策略(Consistent Hashing)

consistent hashing的原理是，首先求出memcached服务器节点的哈希值，并将其配置到0-2的32次方的圆上，然后用同样的方法求出存储数据的键的哈希值，并映射到圆上，然后从数据映射到的位置开始顺时针查找，将数据保存到找到的第一个服务器上，如果超过2的32次方，还没找到，则就保存到第一台服务器上。如下图所示：

![](https://github.com/bingbo/blog/blob/master/images/consistent_hash.png)

相关实现如下部分代码所示：

```c
static mmc_t *mmc_consistent_find(mmc_consistent_state_t *state, unsigned int point) /* {{{ */
{
    int lo = 0, hi = state->num_points - 1, mid;

    while (1) {
        /* point is outside interval or lo >= hi, wrap-around */
        if (point <= state->points[lo].point || point > state->points[hi].point) {
            return state->points[lo].server;
        }

        /* test middle point */
        mid = lo + (hi - lo) / 2;
        MMC_DEBUG(("mmc_consistent_find: lo %d, hi %d, mid %d, point %u, midpoint %u", lo, hi, mid, point, state->points[mid].point));

        /* perfect match */
        if (point <= state->points[mid].point && point > (mid ? state->points[mid-1].point : 0)) {
            return state->points[mid].server;
        }

        /* too low, go up */
        if (state->points[mid].point < point) {
            lo = mid + 1;
        }
        else {
            hi = mid - 1;
        }
    }
}
```


> 余数分布式算法由于保存键的服务器会发生巨大变化而影响缓存命中率，而持久哈希中，只有在增加服务器的地点逆时针方向的第一台服务器上的键会受到影响。

## 计算哈希值的哈希算法

### CRC32

```c
static unsigned int mmc_hash_crc32(const char *key, int key_len) /* CRC32 hash {{{ */
{
    unsigned int crc = ~0;
    int i;

    for (i=0; i<key_len; i++) {
        CRC32(crc, key[i]);
    }

    return ~crc;
}
```

### FNV

```c
static unsigned int mmc_hash_fnv1a(const char *key, int key_len) /* FNV-1a hash {{{ */
{
    unsigned int hval = FNV_32_INIT;
    int i;

    for (i=0; i<key_len; i++) {
        hval ^= (unsigned int)key[i];
        hval *= FNV_32_PRIME;
    }

    return hval;
}
```