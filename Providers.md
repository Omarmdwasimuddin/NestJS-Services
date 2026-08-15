# NestJS Providers 

## Provider আসলে কী?

NestJS-এ **Provider** হলো একটা core concept। Service, repository, factory, helper — এই সবগুলোকেই Provider হিসেবে ধরা যায়।

Provider-এর মূল ধারণাটা হলো — এটাকে একটা **dependency হিসেবে inject করা যায়**। মানে একটা class অন্য একটা class-এর ভেতরে "প্রয়োজন" হিসেবে ঢুকিয়ে দেওয়া যায়, আর এই ঢোকানোর (wiring) কাজটা Nest নিজেই runtime-এ করে দেয় — তোমাকে manually `new` করে object বানাতে হয় না।

**Controller** এর কাজ হলো HTTP request handle করা, আর জটিল কাজগুলো **Provider**-এর কাছে পাঠিয়ে দেওয়া (delegate করা)। অর্থাৎ:

- Controller → request receive করে
- Provider (Service) → আসল logic (data storage, business logic) সামলায়

---

## Service — একটা Provider-এর উদাহরণ

ধরো আমরা একটা `CatsService` বানাচ্ছি, যেটা cat-এর data রাখবে আর দেবে।

```ts
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```

এখানে গুরুত্বপূর্ণ জিনিস হলো `@Injectable()` **decorator**। এই decorator-টা class-এর গায়ে একটা metadata লাগিয়ে দেয়, যার মানে হলো — "এই class-টা Nest-এর **IoC container** manage করতে পারবে।"

> **Tip:** CLI দিয়ে service বানাতে চাইলে কমান্ড: `nest g service cats`

---

## Controller-এ Service ব্যবহার করা

এখন `CatsService`-কে `CatsController`-এর ভেতরে ব্যবহার করা যাক:

```ts
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

লক্ষ্য করো — `constructor(private catsService: CatsService) {}` এই লাইনটায় `private` keyword ব্যবহার করা হয়েছে। এটা একটা shorthand — এতে একই সাথে `catsService` নামের একটা property declare হচ্ছে আর সেটাতে value assign-ও হয়ে যাচ্ছে। আলাদা করে `this.catsService = catsService` লেখা লাগছে না।

---

## Dependency Injection (DI) — এই পুরো ব্যাপারটার নাম

Nest এই যে "constructor-এ লিখে দিলেই সেটা নিজে থেকে এসে যাচ্ছে" — এই পুরো design pattern-টার নাম **Dependency Injection**।

TypeScript-এর type system ব্যবহার করে Nest বুঝে ফেলে কোন dependency লাগবে। যেমন উপরের উদাহরণে:

- Nest দেখে যে `CatsController`-এর `CatsService` লাগবে
- Nest নিজে থেকে `CatsService`-এর একটা instance বানায় (অথবা যদি আগে থেকেই বানানো instance থাকে — singleton হলে — সেটাই ফেরত দেয়)
- সেই instance-টা constructor-এ ঢুকিয়ে দেয়

তোমাকে কোথাও `new CatsService()` লিখতে হয় না — এটাই DI-এর মূল সুবিধা।

---

## Scope — Provider কতক্ষণ বেঁচে থাকে

Provider-এর একটা "lifetime" থাকে, যাকে বলে **scope**।

- সাধারণত application যখন start (bootstrap) হয়, তখন প্রতিটা provider একবার instantiate হয়ে যায় এবং application চলা পর্যন্ত সেটাই ব্যবহৃত হয় (singleton-এর মতো)
- Application বন্ধ হলে সব provider destroy হয়ে যায়
- চাইলে একটা provider-কে **request-scoped** করা যায় — মানে সেটার lifetime একটা নির্দিষ্ট HTTP request-এর সাথে বাঁধা থাকবে, প্রতিটা নতুন request-এ নতুন instance বানানো হবে

---

## Custom Provider

Nest-এর ভেতরে একটা built-in **IoC container** আছে যেটা provider-দের মধ্যেকার সম্পর্ক manage করে। শুধু class ছাড়াও আরও অনেকভাবে provider define করা যায় — যেমন plain value দিয়ে, অথবা synchronous/asynchronous factory function দিয়ে।

---

## Optional Provider

মাঝে মাঝে এমন dependency থাকে যেটা resolve না হলেও সমস্যা নেই — যেমন একটা configuration object, যেটা না দিলে default value ব্যবহার হবে। এরকম ক্ষেত্রে সেই dependency-কে **optional** বলা হয়, আর সেটা না পেলে Nest error দেবে না।

Optional করতে `@Optional()` decorator ব্যবহার করতে হয়:

```ts
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(@Optional() @Inject('HTTP_OPTIONS') private httpClient: T) {}
}
```

এখানে `HTTP_OPTIONS` একটা **custom token** — যেহেতু এটা একটা class না, বরং custom provider, তাই token দিয়ে চিহ্নিত করতে হচ্ছে `@Inject()` দিয়ে।

---

## Property-based Injection

এতক্ষণ যেটা দেখলাম সেটা হলো **constructor-based injection** — constructor-এর মাধ্যমে dependency inject করা। এটাই সবচেয়ে বেশি recommended।

তবে বিশেষ কিছু ক্ষেত্রে **property-based injection** কাজে লাগে। যেমন — যদি একটা top-level (parent) class-এর একাধিক provider লাগে, আর সেগুলো sub-class-এ `super()` দিয়ে পাস করা ঝামেলার হয়ে যায়, তখন সরাসরি property-তে `@Inject()` বসিয়ে দেওয়া যায়:

```ts
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  @Inject('HTTP_OPTIONS')
  private readonly httpClient: T;
}
```

> **সতর্কতা:** যদি তোমার class অন্য কোনো class extend না করে, তাহলে constructor-based injection-ই ব্যবহার করা উচিত। কারণ constructor দেখলেই বোঝা যায় কী কী dependency লাগবে — এটা code readability-র জন্য ভালো।

---

## Provider Registration — Module-এ যোগ করা

Provider বানালেই (যেমন `CatsService`) হবে না — সেটাকে module-এ **register** করতে হবে, যাতে Nest সেটা resolve করতে পারে। এটা করতে হয় `app.module.ts` ফাইলে, `@Module()` decorator-এর `providers` array-তে:

```ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```

এখন Nest জানে যে `CatsController`-এর `CatsService` লাগবে, আর সেটা কোথায় পাবে।

### Directory Structure (সাধারণত এরকম হয়)

```
src
 └── cats
      ├── dto
      │    └── create-cat.dto.ts
      ├── interfaces
      │    └── cat.interface.ts
      ├── cats.controller.ts
      └── cats.service.ts
 ├── app.module.ts
 └── main.ts
```

---

## Manual Instantiation — নিজে থেকে instance বানানো

সাধারণত Nest নিজেই সব dependency resolve করে দেয়। কিন্তু কিছু বিশেষ ক্ষেত্রে built-in DI system-এর বাইরে গিয়ে manually provider retrieve বা instantiate করতে হতে পারে:

1. **Module reference** ব্যবহার করে — চলমান application থেকে existing instance আনতে বা dynamically নতুন provider বানাতে
2. **Standalone applications** — `bootstrap()` function-এর ভেতরে provider ব্যবহার করতে (যেমন bootstrap করার সময়ই একটা config service লাগবে)

---
