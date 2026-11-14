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
import { CanActivate, ExecutionContext, Injectable, BadRequestException } from '@nestjs/common';
import { RedisService } from '../redis/redis.service';

@Injectable()
export class RateLimitGuard implements CanActivate{
    constructor(
        private readonly redisService: RedisService,
    ) {}

    async canActivate(context: ExecutionContext): | Promise<boolean>{
        const request = context.switchToHttp().getRequest();
        const ip = request.ip || 'unknown';
        const key = `rate-limit:${ip}`;
        const currentRequests = await this.redisService.client.incr(key);
        if (currentRequests === 1) {
           await this.redisService.client.expire(
            key,
            60
           ) 
        }
        if (currentRequests > 5) {
            throw new BadRequestException('Too many requests. Please try again later.');
        }
        return true;
    }
} 
```
---


#### product-service/src/app/app.controller.ts
```ts
import { Controller, Get, Param, Patch, UseGuards } from '@nestjs/common';
import { AppService } from './app.service';
import { RateLimitGuard } from '../guards/rate-limit.guard';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get('products/:id')
  async getProduct(@Param('id') id: string) {
    return this.appService.getProducts(Number(id));
  }

  @Patch('products/:id/:price')
  async updateProduct(
    @Param('id') id: string,
    @Param('price') price: string,
  ) {
    return this.appService.updateProduct(Number(id), Number(price));
  }

  @Get('products')
  @UseGuards(
    RateLimitGuard
  )
  getProducts(){
    return {
      success: true,
      products: [
        'Laptop',
        'Tablet',
        'iPhone',
      ]
    }
  }
}
```
---


#### product-service/src/app/app.module.ts
```ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { CacheModule } from '@nestjs/cache-manager';
import { redisStore } from 'cache-manager-redis-store';
import { RedisService } from '../redis/redis.service';
import { RateLimitGuard } from '../guards/rate-limit.guard';

@Module({
  imports: [
    CacheModule.registerAsync({
      isGlobal: true,
      useFactory: async() => ({
        store: await redisStore({
          socket: {
            host: 'localhost',
            port: 6379,
          },
          ttl: 60, // seconds
        }),
      })
    })
  ],
  controllers: [AppController],
  providers: [
    AppService, 
    RedisService, 
    RateLimitGuard
  ],
})
export class AppModule {}
```
---


#### Setup Redis Server on WSL Ubuntu
```bash
wsl -d Ubuntu
```
```bash
sudo systemctl start redis-server
```
```bash
npx nx serve product-service
```
---
