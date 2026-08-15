# NestJS-Services

#### Create service
```bash
$ nest g service [name]
```
#### Or,
```bash
$ nest g s [name]
```
![](https://imgur.com/St2SIDb.png)

---

#### product.service.ts
```bash
import { Injectable } from '@nestjs/common';

@Injectable()
export class ProductService {
    private products = [
        {id: 1, name:"Mobile", price: 15000},
        {id: 2, name:"Laptop", price: 80000},
        {id: 3, name:"Tablet", price: 19000},
        {id: 4, name:"SSD", price: 12000},
        {id: 5, name:"DHR Camera", price: 27000},
    ];
    getAllProducts(){
        return this.products;
    }
    getProductById(id: number){
        return this.products.find((product) => product.id === id)
    }
}
```
---


#### create product controller
```bash
nest g controller product
```
---
