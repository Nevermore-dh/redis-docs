---
title: 'Redis: Leaky-bucket rate-limiting [no dependencies]'
---

<CoverImage title="Leaky-bucket rate-limiting with Redis" />

[go-redis/redis_rate](https://github.com/go-redis/redis_rate) library implements a leaky bucket
scheduling algorithm (AKA generic cell rate algorithm). You can install it with:

```shell
go get github.com/redis/go-redis_rate/v9
```

redis_rate accepts an interface to communicate with Redis and thus supports all flavors of go-redis
clients.

```go
rdb := redis.NewClient(&redis.Options{
    Addr: "localhost:6379",
})


limiter := redis_rate.NewLimiter(rdb)
res, err := limiter.Allow(ctx, "project:123", redis_rate.PerSecond(10))
if err != nil {
    panic(err)
}

fmt.Println("allowed", res.Allowed, "remaining", res.Remaining)
```

The following example demonstrates how to use redis_rate in a
[bunrouter](https://github.com/uptrace/bunrouter/tree/master/example/rate-limiting) middleware for
HTTP rate limiting:

```go
func rateLimit(next bunrouter.HandlerFunc) bunrouter.HandlerFunc {
    return func(w http.ResponseWriter, req bunrouter.Request) error {
        res, err := limiter.Allow(req.Context(), "project:123", redis_rate.PerMinute(10))
        if err != nil {
            return err
        }

        h := w.Header()
        h.Set("RateLimit-Remaining", strconv.Itoa(res.Remaining))

        if res.Allowed == 0 {
            // We are rate limited.

            seconds := int(res.RetryAfter / time.Second)
            h.Set("RateLimit-RetryAfter", strconv.Itoa(seconds))

            // Stop processing and return the error.
            return ErrRateLimited
        }

        // Continue processing as normal.
        return next(w, req)
    }
}
```
