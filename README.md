# GoRBAC Cache Redis

为 [gorbac](https://github.com/kordar/gorbac) 提供的 Redis CacheStore 实现，用于在多实例部署场景下共享 RBAC 快照缓存（items/rules/parents），降低冷启动或缓存失效后的加载成本。

---

## 安装

```bash
go get github.com/kordar/gorbac-cache-redis
```

依赖：
- Go 1.18+
- github.com/kordar/gorbac
- github.com/redis/go-redis/v9

---

## 快速使用

```go
package main

import (
    "time"

    "github.com/kordar/gorbac"
    gorbac_cache_redis "github.com/kordar/gorbac-cache-redis"
    "github.com/redis/go-redis/v9"
)

func main() {
    repo := NewYourAuthRepository()

    rdb := redis.NewClient(&redis.Options{
        Addr: "127.0.0.1:6379",
    })
    store := gorbac_cache_redis.NewRedisCacheStore(rdb)

    service := gorbac.NewRbacServiceWithCacheStore(
        repo,
        true,
        store,
        "myapp",
        10*time.Minute,
    )

    _ = service
}
```

键示例：`myapp:rbac:snapshot`。

---

## 连接模式

- 单机：

```go
redis.NewClient(&redis.Options{ Addr: "127.0.0.1:6379" })
```

- 哨兵：

```go
redis.NewFailoverClient(&redis.FailoverOptions{
    MasterName: "mymaster",
    SentinelAddrs: []string{"10.0.0.1:26379","10.0.0.2:26379"},
})
```

- 集群（推荐 UniversalClient）：

```go
redis.NewClusterClient(&redis.ClusterOptions{
    Addrs: []string{"10.0.0.1:6379","10.0.0.2:6379","10.0.0.3:6379"},
})
```

---

## License

MIT License

