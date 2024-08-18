## Overview
JavaScript implements inheritance by using **objects**. Each object has an internal link to another object called its **prototype**. That prototype object has a prototype of its own, and so on until an object is reached with null as its prototype

#### Inheriting properties
![Description](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQL65jiNXnKsq2Ri1tcvrVHK8LssmBOduFHUQ&s)
```
// keys like a and b are own properties of object o
const o = {
  a: 1,
  b: 2,
  // __proto__ sets the [[Prototype]]. It's specified here
  __proto__: {
    b: 3,
    c: 4,
  },
};

// o.[[Prototype]] has properties b and c.
// o.[[Prototype]].[[Prototype]] is Object.prototype 
// Finally, o.[[Prototype]].[[Prototype]].[[Prototype]] is null.
// This is the end of the prototype chain.
// Tthe full prototype chain looks like:
// { a: 1, b: 2 } ---> { b: 3, c: 4 } ---> Object.prototype ---> null

console.log(o.a); // 1
console.log(o.b); // 2
console.log(o.c); // 4
console.log(o.d); // undefined
```

#### Inheriting methods
In JavaScript, any function can be added to an object in the form of a property. An inherited function acts just as any other property, including property shadowing.

When an inherited function is executed, the value of this points to the inheriting object, not to the prototype object where the function is an own property.
```
const parent = {
  value: 2,
  method() {
    return this.value + 1;
  },
};

console.log(parent.method()); // 3
// When calling parent.method in this case, 'this' refers to parent

const child = {
  __proto__: parent,
};
console.log(child.method()); // 3
// When child.method is called, 'this' refers to child. So when child inherits the method of parent, 
// the property 'value' is sought on child. 

child.value = 4; 
// This shadows the 'value' property on parent.
// The child object now looks like:
// { value: 4, __proto__: { value: 2, method: [Function] } }
```

#### Constructors
Sometimes we need to create many objects of the same type. To create an object type we use an object constructor function.
In the constructor function, this has no value. The value of this will become the new object when a new object is created.
```
function Person(first, last, age, eye) {
  this.firstName = first;
  this.lastName = last;
  this.age = age;
  this.eyeColor = eye;
}

const myFather = new Person("John", "Doe", 50, "blue");
const myMother = new Person("Sally", "Rally", 48, "green");
const mySister = new Person("Anna", "Rally", 18, "green");
const mySelf = new Person("Johnny", "Rally", 22, "green");
```
#### Adding a Method to a Function 
```
myMother.changeName = function (name) {
  this.lastName = name;
}
console.log(myFather.changeName) //undefined
```
#### Adding a Method to a Constructor
```
Person.prototype.changeName = function (name) {
  this.lastName = name;
}
console.log(Object.getPrototypeOf(myMother)===Person.prototype); //true
console.log(Person.prototype.constructor === Person); //true
```

**DO NOT** reassign the constructor prototype
* The [[Prototype]] of instances created before the reassignment will be referencing a different object from the [[Prototype]] of instances created after the reassignment.

#### Monkey Patching
For example avoid adding functions to classes like Array. Why?
Because doing monkey patching risks forward compatibility, because if the language adds this method in the future but with a different signature, your code will break.
```
// Do No Do This
Array.prototype.myMethod = function () {...}
```
