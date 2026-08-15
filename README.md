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
<img width="272" height="112" alt="image" src="https://github.com/user-attachments/assets/abee923c-98a7-40a4-a77e-a0021ea99eef" />

---


#### product.controller.ts
```bash
import { Controller, Param, Get } from '@nestjs/common';
import { ProductService } from './product.service';

@Controller('product')
export class ProductController {
    constructor(private readonly productService: ProductService){}
        @Get()
        getProducts(){
            return this.productService.getAllProducts();
        }
        @Get(':id')
        getProduct(@Param('id') id:string){
            return this.productService.getProductById(Number(id))
        }
    
}
```
---

> ### Output
> <img width="362" height="212" alt="image" src="https://github.com/user-attachments/assets/f5ea9a77-4e25-4d02-97b0-85cbd4e19fef" />
