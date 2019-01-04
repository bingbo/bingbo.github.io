### 针对php扩展的memcache源码下载

```bash
git clone https://github.com/php/pecl-caching-memcache.git
```

### php与memcache分布式交互流程

![](https://github.com/bingbo/blog/blob/master/images/php_memcache.jpg)

### memcache类及函数的定义

> 类及函数的定义在memcache.c文件中

```c
//类中的函数
zend_function_entry memcache_functions[] = {
	PHP_FE(memcache_connect,		NULL)
	PHP_FE(memcache_pconnect,		NULL)
	PHP_FE(memcache_add_server,		NULL)
	PHP_FE(memcache_set_server_params,		NULL)
	PHP_FE(memcache_get_server_status,		NULL)
	PHP_FE(memcache_get_version,	NULL)
	PHP_FE(memcache_add,			NULL)
	PHP_FE(memcache_set,			NULL)
	PHP_FE(memcache_replace,		NULL)
	PHP_FE(memcache_get,			NULL)
	PHP_FE(memcache_delete,			NULL)
	PHP_FE(memcache_debug,			NULL)
	PHP_FE(memcache_get_stats,		NULL)
	PHP_FE(memcache_get_extended_stats,		NULL)
	PHP_FE(memcache_set_compress_threshold,	NULL)
	PHP_FE(memcache_increment,		NULL)
	PHP_FE(memcache_decrement,		NULL)
	PHP_FE(memcache_close,			NULL)
	PHP_FE(memcache_flush,			NULL)
	PHP_FE(memcache_setoptimeout,	NULL)
	{NULL, NULL, NULL}
};

static zend_function_entry php_memcache_class_functions[] = {
	PHP_FALIAS(connect,			memcache_connect,			NULL)
	PHP_FALIAS(pconnect,		memcache_pconnect,			NULL)
	PHP_FALIAS(addserver,		memcache_add_server,		NULL)
	PHP_FALIAS(setserverparams,		memcache_set_server_params,		NULL)
	PHP_FALIAS(getserverstatus,		memcache_get_server_status,		NULL)
	PHP_FALIAS(getversion,		memcache_get_version,		NULL)
	PHP_FALIAS(add,				memcache_add,				NULL)
	PHP_FALIAS(set,				memcache_set,				NULL)
	PHP_FALIAS(replace,			memcache_replace,			NULL)
	PHP_FALIAS(get,				memcache_get,				NULL)
	PHP_FALIAS(delete,			memcache_delete,			NULL)
	PHP_FALIAS(getstats,		memcache_get_stats,			NULL)
	PHP_FALIAS(getextendedstats,		memcache_get_extended_stats,		NULL)
	PHP_FALIAS(setcompressthreshold,	memcache_set_compress_threshold,	NULL)
	PHP_FALIAS(increment,		memcache_increment,			NULL)
	PHP_FALIAS(decrement,		memcache_decrement,			NULL)
	PHP_FALIAS(close,			memcache_close,				NULL)
	PHP_FALIAS(flush,			memcache_flush,				NULL)
	PHP_FALIAS(setoptimeout,	memcache_setoptimeout,		NULL)
	{NULL, NULL, NULL}
};

PHP_MINIT_FUNCTION(memcache)
{
    //初始化Memcache类定义模块
	zend_class_entry memcache_class_entry;
	INIT_CLASS_ENTRY(memcache_class_entry, "Memcache", php_memcache_class_functions);
	memcache_class_entry_ptr = zend_register_internal_class(&memcache_class_entry TSRMLS_CC);
    ......
}
```

### memcache_add_server函数的定义

```c
PHP_FUNCTION(memcache_add_server)
{
	zval **connection, *mmc_object = getThis(), *failure_callback = NULL;
    //缓存连接池
	mmc_pool_t *pool;
    //当前要添加的服务器结构变量
	mmc_t *mmc;
	long port = MEMCACHE_G(default_port), weight = 1, timeout = MMC_DEFAULT_TIMEOUT, retry_interval = MMC_DEFAULT_RETRY, timeoutms = 0;
	zend_bool persistent = 1, status = 1;
	int resource_type, host_len, list_id;
	char *host;

	........

    //如果没有初始化过服务器连接池，则新建一个
	/* lazy initialization of server struct */
	if (persistent) {
		mmc = mmc_find_persistent(host, host_len, port, timeout, retry_interval TSRMLS_CC);
	}
	else {
		MMC_DEBUG(("memcache_add_server: initializing regular struct"));
        //初始化pool
		mmc = mmc_server_new(host, host_len, port, 0, timeout, retry_interval TSRMLS_CC);
	}

    ......

    //将新增服务器添加到pool中
	mmc_pool_add(pool, mmc, weight);
	RETURN_TRUE;
}
```

> mmc_pool_t结构定义

```c
#php_memcache.h
typedef struct mmc_pool {
	mmc_t					**servers;//所有的服务器
	int						num_servers;//服务器数量
	mmc_t					**requests;//根据key返回的服务器
	int						compress_threshold;// 待存储的数据压缩的下限值
	double					min_compress_savings;// 待存储的数据最小的压缩百分比
	zend_bool				in_free;//标记该pool是否被释放
	mmc_hash_t				*hash;//hash策略容器
	void					*hash_state;//hash函数
} mmc_pool_t;
```

> mmc_t结构的定义

```c
#php_memcache.h
typedef struct mmc {
	php_stream				*stream;
	char					inbuf[MMC_BUF_SIZE];
	smart_str				outbuf;
	char					*host;
	unsigned short			port;
	long					timeout;
	long					timeoutms; /* takes precedence over timeout */
	long					connect_timeoutms; /* takes precedence over timeout */
	long					failed;
	long					retry_interval;
	int						persistent;
	int						status;
	char					*error;					/* last error message */
	int						errnum;					/* last error code */
	zval					*failure_callback;
	zend_bool				in_free;
} mmc_t;
```

> mmc_hash_t结构的定义

```c
#php_memcache.h
typedef struct mmc_hash {
	mmc_hash_create_state	create_state;// 创建hash策略状态
	mmc_hash_free_state		free_state;// 释放hash策略状态
	mmc_hash_find_server	find_server;// 根据key和分布式算法定位到某台服务器
	mmc_hash_add_server		add_server;//根据hash策略、算法以及权重值添加服务器资源
} mmc_hash_t;
```

> 创建服务器连接池

```c
#memcache.c
mmc_pool_t *mmc_pool_new(TSRMLS_D) /* {{{ */
{
	mmc_pool_t *pool = emalloc(sizeof(mmc_pool_t));
	pool->num_servers = 0;
	pool->compress_threshold = 0;
	pool->in_free = 0;
	pool->min_compress_savings = MMC_DEFAULT_SAVINGS;

	mmc_pool_init_hash(pool TSRMLS_CC);

	return pool;
}
```

> 接着初始化服务器连接池

```c
#memcache.c
static void mmc_pool_init_hash(mmc_pool_t *pool TSRMLS_DC) /* {{{ */
{
	mmc_hash_function hash;

    //根据php.ini中配置的memcache.hash_strategy选择hash存储策略,默认为标准hash存储策略
	switch (MEMCACHE_G(hash_strategy)) {
		case MMC_CONSISTENT_HASH:
			pool->hash = &mmc_consistent_hash;//采用持久化hash存储策略
			break;
		default:
			pool->hash = &mmc_standard_hash;//采用标准hash存储策略
	}

    //根据php.ini中的memcache.hash_function配置选择hash函数，默认为crc32算法
	switch (MEMCACHE_G(hash_function)) {
		case MMC_HASH_FNV1A:
			hash = &mmc_hash_fnv1a;//采用fnv1a算法
			break;
		default:
			hash = &mmc_hash_crc32;//采用crc32算法
	}
	//hash策略中根据选择的hash函数创建对应的状态
	pool->hash_state = pool->hash->create_state(hash);
}
```

> 这里有两种不同的hash存储算法，一种是mmc_standard_hash函数结构，一种是mmc_consistent_hash函数结构，先看下标准的hash存储策略

```c
#memcache_standard_hash.c
//标准hash策略所包含的函数结构
mmc_hash_t mmc_standard_hash = {
	mmc_standard_create_state,
	mmc_standard_free_state,
	mmc_standard_find_server,
	mmc_standard_add_server
};

//标准hash策略状态
typedef struct mmc_standard_state {
	int						num_servers;
	mmc_t					**buckets;
	int						num_buckets;
	mmc_hash_function		hash;
} mmc_standard_state_t;

void *mmc_standard_create_state(mmc_hash_function hash) /* {{{ */
{
    //初始化状态
	mmc_standard_state_t *state = emalloc(sizeof(mmc_standard_state_t));
	memset(state, 0, sizeof(mmc_standard_state_t));
    //选择的hash函数赋值给hash属性
	state->hash = hash;
	return state;
}
/* }}} */


```

> 同样的两种hash储存算法CRC32及fnv1a等，如下所示：

```c
#memcache.c
static unsigned int mmc_hash_crc32(const char *key, int key_len) /* CRC32 hash {{{ */
{
	unsigned int crc = ~0;
	int i;

	for (i=0; i<key_len; i++) {
		CRC32(crc, key[i]);
	}

  	return ~crc;
}
/* }}} */

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

> 下面是持久化hash策略相应结构及实现

```c
#memcache_consistent_hash.c
typedef struct mmc_consistent_point {
	mmc_t					*server;//当前服务器
	unsigned int			point;//对应的指针
} mmc_consistent_point_t;

typedef struct mmc_consistent_state {
	int						num_servers;//服务器数量
	mmc_consistent_point_t	*points;//服务器指针
	int						num_points;//指针数量
	mmc_t					*buckets[MMC_CONSISTENT_BUCKETS];//哈希桶
	int						buckets_populated;//标记哈希桶是否计算过
	mmc_hash_function		hash;//哈希函数
} mmc_consistent_state_t;

void *mmc_consistent_create_state(mmc_hash_function hash) /* {{{ */
{
	mmc_consistent_state_t *state = emalloc(sizeof(mmc_consistent_state_t));
	memset(state, 0, sizeof(mmc_consistent_state_t));
	state->hash = hash;
	return state;
}
/* }}} */

mmc_hash_t mmc_consistent_hash = {
	mmc_consistent_create_state,
	mmc_consistent_free_state,
	mmc_consistent_find_server,
	mmc_consistent_add_server
};

> 添加服务器到服务器连接池中

```c
#memcache.c
void mmc_pool_add(mmc_pool_t *pool, mmc_t *mmc, unsigned int weight) /* {{{ */
{
	/* add server and a preallocated request pointer */
	if (pool->num_servers) {
		pool->servers = erealloc(pool->servers, sizeof(mmc_t *) * (pool->num_servers + 1));
		pool->requests = erealloc(pool->requests, sizeof(mmc_t *) * (pool->num_servers + 1));
	}
	else {
		pool->servers = emalloc(sizeof(mmc_t *));
		pool->requests = emalloc(sizeof(mmc_t *));
	}

	pool->servers[pool->num_servers] = mmc;
	pool->num_servers++;

    //根据相应的hash策略状态来添加服务器到池中并设置相应的权重
	pool->hash->add_server(pool->hash_state, mmc, weight);
}
```

> add_server会调用相应hash策略的添加函数，如

_持久hash策略_

```c
#memcache_consistent_hash.c
void mmc_consistent_add_server(void *s, mmc_t *mmc, unsigned int weight) /* {{{ */
{
	mmc_consistent_state_t *state = s;
	int i, key_len, points = weight * MMC_CONSISTENT_POINTS;

	/* buffer for "host:port-i\0" */
	char *key = emalloc(strlen(mmc->host) + MAX_LENGTH_OF_LONG * 2 + 3);

	/* add weight * MMC_CONSISTENT_POINTS number of points for this server */
	state->points = erealloc(state->points, sizeof(mmc_consistent_point_t) * (state->num_points + points));

	for (i=0; i<points; i++) {
		key_len = sprintf(key, "%s:%d-%d", mmc->host, mmc->port, i);
		state->points[state->num_points + i].server = mmc;
		state->points[state->num_points + i].point = state->hash(key, key_len);
		MMC_DEBUG(("mmc_consistent_add_server: key %s, point %lu", key, state->points[state->num_points + i].point));
	}

	state->num_points += points;
	state->num_servers++;
	state->buckets_populated = 0;

	efree(key);
}
```

_标准hash策略_

```c
void mmc_standard_add_server(void *s, mmc_t *mmc, unsigned int weight) /* {{{ */
{
	mmc_standard_state_t *state = s;
	int i;

	/* add weight number of buckets for this server */
	if (state->num_buckets) {
		state->buckets = erealloc(state->buckets, sizeof(mmc_t *) * (state->num_buckets + weight));
	}
	else {
		state->buckets = emalloc(sizeof(mmc_t *) * (weight));
	}

	for (i=0; i<weight; i++) {
		state->buckets[state->num_buckets + i] = mmc;
	}

	state->num_buckets += weight;
	state->num_servers++;
}
```

### 添加数据set()

> memcache_set函数定义

```c
#memcache.c
PHP_FUNCTION(memcache_set)
{
	php_mmc_store(INTERNAL_FUNCTION_PARAM_PASSTHRU, "set", sizeof("set") - 1);
}
```

> php_mmc_store函数的实现

```c
#memcache.c
static void php_mmc_store(INTERNAL_FUNCTION_PARAMETERS, char *command, int command_len) /* {{{ */
{
	mmc_pool_t *pool;
	zval *value, *mmc_object = getThis();

	int result, key_len;
	char *key;
	long flags = 0, expire = 0;
	char key_tmp[MMC_KEY_MAX_SIZE];
	unsigned int key_tmp_len;

	php_serialize_data_t value_hash;
	smart_str buf = {0};

	..........
	
    //获取服务器连接池
	if (!mmc_get_pool(mmc_object, &pool TSRMLS_CC) || !pool->num_servers) {
		RETURN_FALSE;
	}

    //根据不同的数据类型进行不同的储存处理
	switch (Z_TYPE_P(value)) {
		case IS_STRING:
			result = mmc_pool_store(
				pool, command, command_len, key_tmp, key_tmp_len, flags, expire, 
				Z_STRVAL_P(value), Z_STRLEN_P(value) TSRMLS_CC);
			break;

		case IS_LONG:
		case IS_DOUBLE:
		case IS_BOOL: {
			zval value_copy;

			/* FIXME: we should be using 'Z' instead of this, but unfortunately it's PHP5-only */
			value_copy = *value;
			zval_copy_ctor(&value_copy);
			convert_to_string(&value_copy);

			result = mmc_pool_store(
				pool, command, command_len, key_tmp, key_tmp_len, flags, expire, 
				Z_STRVAL(value_copy), Z_STRLEN(value_copy) TSRMLS_CC);

			zval_dtor(&value_copy);
			break;
		}

		default: {
			zval value_copy, *value_copy_ptr;

			/* FIXME: we should be using 'Z' instead of this, but unfortunately it's PHP5-only */
			value_copy = *value;
			zval_copy_ctor(&value_copy);
			value_copy_ptr = &value_copy;

			PHP_VAR_SERIALIZE_INIT(value_hash);
			php_var_serialize(&buf, &value_copy_ptr, &value_hash TSRMLS_CC);
			PHP_VAR_SERIALIZE_DESTROY(value_hash);

			if (!buf.c) {
				/* something went really wrong */
				zval_dtor(&value_copy);
				php_error_docref(NULL TSRMLS_CC, E_WARNING, "Failed to serialize value");
				RETURN_FALSE;
			}

			flags |= MMC_SERIALIZED;
			zval_dtor(&value_copy);

			result = mmc_pool_store(
				pool, command, command_len, key_tmp, key_tmp_len, flags, expire, 
				buf.c, buf.len TSRMLS_CC);
		}
	}

	......
}
```

> mc_pool_store函数实现

```c
#memcache.c
int mmc_pool_store(mmc_pool_t *pool, const char *command, int command_len, const char *key, int key_len, int flags, int expire, const char *value, int value_len TSRMLS_DC) /* {{{ */
{

	.........
    //通过key确定服务器
	while (result < 0 && (mmc = mmc_pool_find(pool, key, key_len TSRMLS_CC)) != NULL) {
		if ((result = mmc_server_store(mmc, request, request_len TSRMLS_CC)) < 0) {
			mmc_server_failure(mmc TSRMLS_CC);
		}
	}

	if (key_copy != NULL) {
		efree(key_copy);
	}

	if (data != NULL) {
		efree(data);
	}

	efree(request);

	return result;
}
```

> mmc_pool_find函数

```c
#php_memcache.h

#define mmc_pool_find(pool, key, key_len) \
	pool->hash->find_server(pool->hash_state, key, key_len)
```

> find_server在标准hash模式中的函数为mmc_standard_find_server，在持久化hash模式中的函数为mmc_consistent_find_server

_mmc_standard_find_server_

```c
#memcache_standard_hash.c
mmc_t *mmc_standard_find_server(void *s, const char *key, int key_len TSRMLS_DC) /* {{{ */
{
	mmc_standard_state_t *state = s;
	mmc_t *mmc;

	if (state->num_servers > 1) {
		unsigned int hash = mmc_hash(state, key, key_len), i;
		mmc = state->buckets[hash % state->num_buckets];

		/* perform failover if needed */
		for (i=0; !mmc_open(mmc, 0, NULL, NULL TSRMLS_CC) && MEMCACHE_G(allow_failover) && i<MEMCACHE_G(max_failover_attempts); i++) {
			char *next_key = emalloc(key_len + MAX_LENGTH_OF_LONG + 1);
			int next_len = sprintf(next_key, "%d%s", i+1, key);
			MMC_DEBUG(("mmc_standard_find_server: failed to connect to server '%s:%d' status %d, trying next", mmc->host, mmc->port, mmc->status));

			hash += mmc_hash(state, next_key, next_len);
			mmc = state->buckets[hash % state->num_buckets];

			efree(next_key);
		}
	}
	else {
		mmc = state->buckets[0];
		mmc_open(mmc, 0, NULL, NULL TSRMLS_CC);
	}

	return mmc->status != MMC_STATUS_FAILED ? mmc : NULL;
}
```

_mmc_consistent_find_server_

```c
#memcache_consistent_hash.c
mmc_t *mmc_consistent_find_server(void *s, const char *key, int key_len TSRMLS_DC) /* {{{ */
{
	mmc_consistent_state_t *state = s;
	mmc_t *mmc;

	if (state->num_servers > 1) {
		unsigned int i, hash = state->hash(key, key_len);

		if (!state->buckets_populated) {
			mmc_consistent_populate_buckets(state);
		}

		mmc = state->buckets[hash % MMC_CONSISTENT_BUCKETS];

		/* perform failover if needed */
		for (i=0; !mmc_open(mmc, 0, NULL, NULL TSRMLS_CC) && MEMCACHE_G(allow_failover) && i<MEMCACHE_G(max_failover_attempts); i++) {
			char *next_key = emalloc(key_len + MAX_LENGTH_OF_LONG + 1);
			int next_len = sprintf(next_key, "%s-%d", key, i);
			MMC_DEBUG(("mmc_consistent_find_server: failed to connect to server '%s:%d' status %d, trying next", mmc->host, mmc->port, mmc->status));

			hash = state->hash(next_key, next_len);
			mmc = state->buckets[hash % MMC_CONSISTENT_BUCKETS];

			efree(next_key);
		}
	}
	else {
		mmc = state->points[0].server;
		mmc_open(mmc, 0, NULL, NULL TSRMLS_CC);
	}

	return mmc->status != MMC_STATUS_FAILED ? mmc : NULL;
}
```

### 获取数据get

> memcache_get函数定义

```c
#memcache.c
PHP_FUNCTION(memcache_get)
{
    .....
    //获取相应的服务器
	if (!mmc_get_pool(mmc_object, &pool TSRMLS_CC) || !pool->num_servers) {
		RETURN_FALSE;
	}

	if (Z_TYPE_P(zkey) != IS_ARRAY) {
        //检查key的合法性
		if (mmc_prepare_key(zkey, key, &key_len TSRMLS_CC) == MMC_OK) {
            //获取key的value
			if (mmc_exec_retrieval_cmd(pool, key, key_len, &return_value, flags TSRMLS_CC) < 0) {
				zval_dtor(return_value);
				RETVAL_FALSE;
			}
		}
		else {
			RETVAL_FALSE;
		}
	} else if (zend_hash_num_elements(Z_ARRVAL_P(zkey))){
		if (mmc_exec_retrieval_cmd_multi(pool, zkey, &return_value, flags TSRMLS_CC) < 0) {
			zval_dtor(return_value);
			RETVAL_FALSE;
		}
	} else {
		RETVAL_FALSE;
	}
}

> mmc_prepare_key函数实现

```c
#memcache.c
int mmc_prepare_key(zval *key, char *result, unsigned int *result_len TSRMLS_DC)  /* {{{ */
{
	if (Z_TYPE_P(key) == IS_STRING) {
		return mmc_prepare_key_ex(Z_STRVAL_P(key), Z_STRLEN_P(key), result, result_len TSRMLS_CC);
	} else {
		int res;
		zval *keytmp;
		ALLOC_ZVAL(keytmp);

		*keytmp = *key;
		zval_copy_ctor(keytmp);
		convert_to_string(keytmp);

		res = mmc_prepare_key_ex(Z_STRVAL_P(keytmp), Z_STRLEN_P(keytmp), result, result_len TSRMLS_CC);

		zval_dtor(keytmp);
		FREE_ZVAL(keytmp);
		
		return res;
	}
}
```

> mmc_exec_retrieval_cmd函数实现

```c
#memcache.c
int mmc_exec_retrieval_cmd(mmc_pool_t *pool, const char *key, int key_len, zval **return_value, zval *return_flags TSRMLS_DC) /* {{{ */
{
	mmc_t *mmc;
	char *command, *value;
	int result = -1, command_len, response_len, value_len, flags = 0;

	MMC_DEBUG(("mmc_exec_retrieval_cmd: key '%s'", key));

	command_len = spprintf(&command, 0, "get %s", key);

	while (result < 0 && (mmc = mmc_pool_find(pool, key, key_len TSRMLS_CC)) != NULL) {
		MMC_DEBUG(("mmc_exec_retrieval_cmd: found server '%s:%d' for key '%s'", mmc->host, mmc->port, key));

		/* send command and read value */
		if ((result = mmc_sendcmd(mmc, command, command_len TSRMLS_CC)) > 0 &&
			(result = mmc_read_value(mmc, NULL, NULL, &value, &value_len, &flags TSRMLS_CC)) >= 0) {

			/* not found */
			if (result == 0) {
				ZVAL_FALSE(*return_value);
			}
			/* read "END" */
			else if ((response_len = mmc_readline(mmc TSRMLS_CC)) < 0 || !mmc_str_left(mmc->inbuf, "END", response_len, sizeof("END")-1)) {
				mmc_server_seterror(mmc, "Malformed END line", 0);
				result = -1;
			}
			else if (flags & MMC_SERIALIZED ) {
				result = mmc_postprocess_value(return_value, value, value_len TSRMLS_CC);				
			}
			else {
				ZVAL_STRINGL(*return_value, value, value_len, 0);
			}
		}

		if (result < 0) {
			mmc_server_failure(mmc TSRMLS_CC);
		}
	}

	if (return_flags != NULL) {
		zval_dtor(return_flags);
		ZVAL_LONG(return_flags, flags);
	}
	
	efree(command);
	return result;
}
```

### 删除数据delete()

> memcache_delete函数定义

```c
#memcache.c
PHP_FUNCTION(memcache_delete)
{
	mmc_t *mmc;
	mmc_pool_t *pool;
	int result = -1, key_len;
	zval *mmc_object = getThis();
	char *key;
	long time = 0;
	char key_tmp[MMC_KEY_MAX_SIZE];
	unsigned int key_tmp_len;

	.....

    //获取服务池
	if (!mmc_get_pool(mmc_object, &pool TSRMLS_CC) || !pool->num_servers) {
		RETURN_FALSE;
	}

    //检查key
	if (mmc_prepare_key_ex(key, key_len, key_tmp, &key_tmp_len TSRMLS_CC) != MMC_OK) {
		RETURN_FALSE;
	}

    //获取相应的服务器
	while (result < 0 && (mmc = mmc_pool_find(pool, key_tmp, key_tmp_len TSRMLS_CC)) != NULL) {
        //删除key
		if ((result = mmc_delete(mmc, key_tmp, key_tmp_len, time TSRMLS_CC)) < 0) {
			mmc_server_failure(mmc TSRMLS_CC);
		}
	}

	if (result > 0) {
		RETURN_TRUE;
	}
	RETURN_FALSE;
}
```

> mmc_delete函数实现

```
#memcache.c
int mmc_delete(mmc_t *mmc, const char *key, int key_len, int time TSRMLS_DC) /* {{{ */
{
	char *command;
	int command_len, response_len;

	command_len = spprintf(&command, 0, "delete %s %d", key, time);

	MMC_DEBUG(("mmc_delete: trying to delete '%s'", key));

	if (mmc_sendcmd(mmc, command, command_len TSRMLS_CC) < 0) {
		efree(command);
		return -1;
	}
	efree(command);

	if ((response_len = mmc_readline(mmc TSRMLS_CC)) < 0){
		MMC_DEBUG(("failed to read the server's response"));
		return -1;
	}

	MMC_DEBUG(("mmc_delete: server's response is '%s'", mmc->inbuf));

	if(mmc_str_left(mmc->inbuf,"DELETED", response_len, sizeof("DELETED") - 1)) {
		return 1;
	}

	if(mmc_str_left(mmc->inbuf,"NOT_FOUND", response_len, sizeof("NOT_FOUND") - 1)) {
		return 0;
	}

	mmc_server_received_error(mmc, response_len);
	return -1;
}
```