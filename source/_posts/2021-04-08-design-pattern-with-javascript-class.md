---
layout: post
title: "JavaScript 设计模式与类"
date: 2021-04-08 22:00:00
comments: true
categories: 
- Document
tags:
- JavaScript
- 设计模式
- 基础知识
---

本文主要针对 JavaScript 类的各种设计模式实现进行简单总结和举例。

### 字面量

```javascript
var BMW = {
	type: 'Car',
	color: 'White',
	run: function () {
		console.log('BMW start!');
	}
}
```

<!-- more -->

### 工厂模式

工厂模式是一种创建型模式，相当于一个封闭的工厂，你只需要通过一个统一接口传入原料，工厂会根据你传入的原料来创建新的对象。工厂模式**不显示使用 new 来进行实例化**。工厂模式不会显式地暴露对象创建的流程和逻辑。

#### **简单工厂模式（静态工厂模式）**

通过一个静态方法来创建对象，只需要传入指定的参数就可以生产对应的产品，而不需要知道具体的生产细节和逻辑判断。

```javascript
// 输入生成对象的原料，可以黑盒生成对象，其中也可以增加一些不可见的逻辑
function vehicleFactory (type, color) {
	function Vehicle (type, color) {
		this.type = type;
		this.color = color;
	}
	var vehicle = new Vehicle(type, color);

	if (type === 'BMW') {
		vehicle.run = function () {
			console.log('BMW start!');
		}
	} else if (type === 'Benz') {
		vehicle.talk = function () {
			console.log('Benz start!');
		}
	}
	return vehicle;
}

var bmw = vehicleFactory('BMW', 'white');
```

```javascript
// 用 TypeScript 类实现
abstract class Vehicle {
  abstract run(): void;
}
class BMW extends Vehicle {
  run(): void {
    console.log('BMW start!');
  }
}
class Benz extends Vehicle {
  run(): void {
    console.log('Benz start!');
  }
}
class VehicleFactory {
  public static produceVehicle(model: "BMW" | "Benz"): Vehicle {
    if (model === "BMW") {
      return new BMW();
    } else {
      return new Benz();
    }
  }
}
let benz = VehicleFactory.produceVehicle('BMW');
```

简单工厂模式将对象的创建和使用进行了解耦（用户不需要关心对象时怎样创建的），并且用户不需要知道产品的所有信息。缺点是系统扩展困难，如果要添加新产品就要修改工厂逻辑。

#### **工厂方法模式**

是简单工厂模式的升级，也被叫做多台工厂模式。工厂方法模式则是工厂将产品信息判断后，派发给下属分工厂进行生产。

```javascript
var VehicleFactory = function (type, color) {
  if (this instanceof VehicleFactory) {
		var vehicle = new this[type](color);
		return vehicle;
	} else  {
		return new VehicleFactory(color);
	}
}

VehicleFactory.prototype = {
	BMW: function (color) {
		this.color = color;
		this.run = function () {
			console.log('BMW start!');
		}
	},
	Benz: function (color) {
		this.color = color;
		this.run = function () {
			console.log('Benz start!');
		}
	}
}

var bmw = new VehicleFactory('BMW', 'white')
```

```javascript
abstract class Vehicle {
	abstract run(): void;
}
class BMW extends Vehicle {
  run(): void {
    console.log("BMW 发动咯");
  }
}
class Benz extends Vehicle {
  run(): void {
    console.log("Benz 发动咯");
  }
}
interface vehicleFactory {
  produceVehicle(): Vehicle;
}
class BMWFactory implements vehicleFactory {
  produceVehicle(): Vehicle {
    return new BMW();
  }
}
class BenzFactory implements vehicleFactory {
  produceVehicle(): Vehicle {
    return new Benz();
  }
}
const bmwFactory = new BMWFactory();
const bmwCar = bmwFactory.produceVehicle;
```

#### **抽象工厂模式**

通过类的抽象使得业务适用于一个产品类粗的创建，而不负责某一类产品的实例。

```javascript
const VehicleFactory = function(subType, superType) {
  if (typeof VehicleFactory[superType] === 'function') {
    function F() {
      this.type = '车辆'
    } 
    F.prototype = new VehicleFactory[superType]()
    subType.constructor = subType
    subType.prototype = new F()                // 因为子类subType不仅需要继承superType对应的类的原型方法，还要继承其对象属性
  } else throw new Error('不存在该抽象类')
}

VehicleFactory.Car = function() {
  this.type = 'car'
}
VehicleFactory.Car.prototype = {
  getPrice: function() {
    return new Error('抽象方法不可使用')
  },
  getSpeed: function() {
    return new Error('抽象方法不可使用')
  }
}

const BMW = function(price, speed) {
  this.price = price
  this.speed = speed
}
VehicleFactory(BMW, 'Car')        // 继承Car抽象类
BMW.prototype.getPrice = function() {        // 覆写getPrice方法
  console.log(`BWM price is ${this.price}`)
}
BMW.prototype.getSpeed = function() {
  console.log(`BWM speed is ${this.speed}`)
}

const baomai5 = new BMW(30, 99)
baomai5.getPrice()                          // BWM price is 30
baomai5 instanceof VehicleFactory.Car
```

### 构造函数（构造器）模式

```javascript
function Car (model, year, miles) {
	this.model = model;
	this.year = year;
	this.miles = miles;
	this.toString = function() {
		return this.model + " has done " + this.miles + " miles";
	}
}

var civic = new Car("Honda Civic", 2009, 20000);
```

想要使用构造函数生成对象实例，必须使用 new 操作符，new 操作符本质上会实现以下几个步骤

- 创建一个空对象： `var obj = new Object()`
- 设置原型链（实例的内部将包含一个指针（内部属性），指向构造函数的原型对象）： `obj.__proto__ = Car.prototype`
- 让 Func 中的 this 指向 obj ，并执行 Func 的函数体： `var res = Car.call(this, ...arguments)`
- 判断 Func 的返回值类型：如果是值类型，返回obj。如果是引用类型，就返回这个引用类型的对象：
    
    ```javascript
    if (typeof(res) == "object"){
      return res;
    }
    else{
      return obj;
    }
    ```
    

### 原型模式

```javascript
function Car () {};
Car.prototype = {
	model: 'Honda Civic',
	year: 2009,
	miles: 20000,
	toString: function() {
			return this.model + " has done " + this.miles + " miles";
	}
}
```

### 组合模式（构造+原型）

```javascript
function Car (model, year, miles) {
	this.model = model;
	this.year = year;
	this.miles = miles;
}

Car.prototype.toString = function() {
	return this.model + " has done " + this.miles + " miles";
}
```

### 动态原型模式

```javascript
function Car(model, year, miles) {
  this.model = model;
	this.year = year;
	this.miles = miles;
  if (typeof this.getModelName != 'function') {
    Car.prototype.getModelName = function() {
      alert(this.name);
    }
  }
}
```

### 寄生构造函数模式

```javascript
function createCar(model, year, miles) {
  var car = new Object();
  car.model = model;
  car.year = year;
  car.miles = miles;
  car.getModelName = function() {
    alert(this.model);
  }
  return car;
}
var p1 = new createCar('Honda Civic', 2009, 20000)
```

### 稳妥构造函数模式

没有公共属性的工厂模式

```javascript
function createCar(model, year, miles) {
  var car = new Object();
  car.getModelName = function() {
    alert(this.model);
  }
  return car;
}
```