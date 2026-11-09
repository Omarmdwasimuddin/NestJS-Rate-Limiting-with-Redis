## NestJS-Rate-Limiting-with-Redis

#### ei project agee koro--> [Redis Caching in NestJS Microservices with Nx (Cache-Aside & Write-Through Patterns)](https://github.com/Omarmdwasimuddin/Redis-Cache-and-Caching-Patterns) karon ei project er workspace ei kaj korbo

#### file-folder banaw: `product-service/src/redis/redis.service.ts`
```ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import { createClient } from 'redis';


@Injectable()
export class RedisService implements OnModuleInit {
    public client = createClient({
        url:'redis://localhost:6379',
    })
    async onModuleInit() {
        await this.client.connect();
        console.log('Redis Connected Successfully!')
    }
    
}
```
---

#### file-folder banaw: `product-service/src/guards/rate-limit.guard.ts`
```ts

```
---


