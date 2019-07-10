---
title: "Javascript tips"
categories:
  - javascript
tags:
  - tips
  - js
  - oop
---

Hello, in this post we'll see some tips and rules that we tend to use in __JS__. 

Today javascript is the most popular programming language
(__PHP fans hate intensifies__), in Backend / Frontend, with amazing frameworks (React, Vue, Express, etc.).

*So why JS and where to begin?*

The [JS documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript) is IMO the perfect documentation: 
* Examples
* Clear explanation
* Forums
* Opensource etc. 

It is the most important stone to build a language knowledge (community !).
We won't be seeing every single angle of it, because internet is providing much more than needed ;). Thus I will show you some fews tips and 
concepts of the pure JS.

## 1) Random tips & knowledge:

```javascript
let x = null;
let y = undefined;
```

In this example x was initialized but has no reference. y exists but has no value. We can say that `let y = undefined;` is equivalent to `let y;`.

Thus `x === undefined` will return **false** (very logic to me).

----

> Next ! 

In order to be seen in a browser, any variables in JS will be converted into a string (implicit conversion).

Mmmm? An example will clarify the statement above :

```javascript
/* p represents a block */
let x = 5;
p.innerHTML = x;
```

Line 1 : x is a number. 

Line 2 : x is converted into a string to be shown in the browser (in the <p></p> block). 

----

> Next ! 

`parseInt()` method converts a string into a number.
  
But it has obviously its limits : 
  
`parseInt('2018')` will return 2018
  
and `parseInt('lololewl')` will return ... `NaN` (Not a Number).
  

----

> Next !

```javascript
if(x){
foo();
}
```
If x is true, then `foo()` will be returned. Nothing new... In JS we can use a [short circuit evaluation](https://en.wikipedia.org/wiki/Short-circuit_evaluation) : `x && foo()`.

Why? 
> As logical expressions are evaluated left to right, they are tested for possible "short-circuit" evaluation

In the AND case, if the left side of the evaluation is true, the result will always depend on the right side, thus in our case : foo(); 

If x is FALSE then no matter what can return foo() (ex: 'Captain Marvel the presomptuous', etc.), the result will be... **FALSE**.

It doesn't even need to evaluate the foo() part ( too bad for you Captain Marvel). 

----

## 2) Functions and Prototypes:

A little tip about functions : 

```javascript
const smash = "Hulk smashed ";

function whatDidHulkDo (name, size, ...args){
    let smash = "Hulk smashed ";
    console.log(args.length);
    return smash + name;
}
let myHero = whatDidHulkDo("thor", 195, "fat");
```

Some are not familiar with the `...args`. It looks like "Arghhh" but it is not. It means you can pass by **any random bonus parameter** to your function. 

In this case, this little function will return : `Hulk smashed thor`, and the console will output : 1, which represents args.length (in this case, `args[0]` will return `"fat"`).

----

> Next !

The last part of this post. Javascript is a **prototype-based** language. Java/PHP are **class-based**.

We can explain inheritanceSection through this small example : 

Let's create a Hero class. 

```javascript
function Hero (name, shape, isDead){
    this.name = name;
    this.shape = shape;
    this.isDead = isDead;
    this.heroDetail = function() {
        return `${this.name.charAt(0).toUpperCase() + this.name.slice(1)} looks ${this.shape} and 
            ${(isDead) ? "is dead" : "is still alive"}.`;
    }
}
```
we can create any kind of Hero, for example : 

```javascript
let nedStark = new Hero("ned", "good", 1);
console.log(nedStark.heroDetail());
```

The output will be : 

	Ned looks good and is dead.

Now, inheritance can be created by prototyping. Imagine we want now a "sub-class" MarvelHero.

```javascript
function MarvelHero (location, strenght){
    this.location = location;
    this.strenght = strenght;
}
MarvelHero.prototype = new Hero();

let thor = new MarvelHero("earth", 999);
```

Thanks to the `MarvelHero.prototype = new Hero();` line, our `thor` object will have also the **Hero** properties. 

So we can do something like : 

```javascript
thor.name = "thor";
thor.shape = "fat";
thor.isDead = 0;
```

We can even add external functions to our existing object, that use extended properties. 

```javascript
const smash = "Hulk smashed ";

MarvelHero.prototype.whatDidHulkDo = function (){
  	this.isDead = 1;
    return `${smash}${this.name}.`;
}
console.log(thor.whatDidHulkDo());
```

The `MarvelHero` object "extends" the `Hero` object, by the famous `MarvelHero.prototype = new Hero();`, then it gets the `name` and `shape` and `isDead` properties in addition to the MarvelHero properties.

What is the output ? 

```javascript
console.log(thor.whatDidHulkDo());
/* OUTPUT : Hulk smashed thor. */

console.log(thor.heroDetail());
/* OUTPUT: Thor looks fat and is dead. */
```

Here ends this mediocre post. Next one will be about Quantum Computing or ReactJS. 
