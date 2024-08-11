1、注意设置过期时间





2、缓存数据库一致性

- 先写数据库，然后再删除缓存
- 要确保数据库与缓存操作的原子性
- 缓存命中则直接返回
- 缓存未命中则查询数据库，并写入缓存，设定超时时间
- 读操作
- 写操作

```java
@Override
public Result queryById(Long id) {
    //从redis查询缓存
    String key = RedisConstants.CACHE_SHOP_KEY + id;
    String shopInfo = stringRedisTemplate.opsForValue().get(key);
    //判断是否存在
    if (StrUtil.isNotBlank(shopInfo)){
        //存在则返回
        Shop shop = JSONUtil.toBean(shopInfo, Shop.class);
        return Result.ok(shop);
    }

    //不存在，则查询数据库
    Shop shop = getById(id);
    //不存在则返回错误
    if (null == shop){
        return Result.fail("店铺不存在");
    }

    //写入缓存 设置超时时间为30min
    stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(shop), RedisConstants.CACHE_SHOP_TTL, TimeUnit.MINUTES);
    //数据库存在，则返回

    return Result.ok(shop);
}

```







```java
@Override
@Transactional
public Result update(Shop shop) {	
    Long id = shop.getId();
    if (null == id){
        return Result.fail("店铺id不能为空");
    }
    //1.更新数据库
    updateById(shop);
    //2.删除缓存
    stringRedisTemplate.delete(RedisConstants.CACHE_SHOP_KEY + shop.getId());
    return Result.ok();
}
```























