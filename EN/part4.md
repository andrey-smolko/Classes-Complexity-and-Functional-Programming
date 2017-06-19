# Appendix

Here are a few extras for your viewing pleasure :)

## The Module Pattern

Another way to avoid the complexities of `this` and leverages simple objects and functions is the Module pattern. You can learn more about this pattern from Addy Osmani’s [“Learning JavaScript Design Patterns”](https://addyosmani.com/resources/essentialjsdesignpatterns/book/) book which is available to read for free [here](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript). Here’s an implementation of our `person` class based on Addy’s “Revealing Module Pattern”:

```js
function getPerson(initialName) {
  let name = initialName
  const person = {
    setName(strName) {
      name = strName
    },
    greeting: 'Hey there!',
    getName() {
      return getPrefixedName('Name')
    },
    getGreetingCallback() {
      const {greeting} = person
      return (subject) => `${greeting} ${subject}, I'm ${name}`
    },
  }
  function getPrefixedName(prefix) {
    return `${prefix}: ${name}`
  }
  return person
}

const person = getPerson('Jane Doe')
person.setName('Sarah Doe')
person.greeting = 'Hello'
person.getName() // Name: Sarah Doe
person.getGreetingCallback()('Jeff') // Hello Jeff, I'm Sarah Doe
```

What I love about this is that there are few concepts to understand. We have a function which creates a few variables and returns an object — simple. Pretty much just objects and functions. For reference, this is what the person object looks like if you expand it in Chrome DevTools:

![pic1](https://cdn-images-1.medium.com/max/800/1*qnMDnUmvNaMRuU2ic9ctOQ.png)

One of the flaws of the module pattern above is that every `person` has its very own copy of each property and function For example:

```js
const person1 = getPerson('Jane Doe')
const person2 = getPerson('Jane Doe')
person1.getGreetingCallback === person2.getGreetingCallback // false
```

Even though the contents of the `getGreetingCallback` function are identical, they will each have their own copy of that function in memory. Most of the time this doesn’t matter, but if you’re planning on making a ton of instances of these, or you want creating these to be more than fast, this can be a bit of a problem. With our `Person` class, every instance we create will have a reference to the exact same method `getGreetingCallback`:

```js
const person1 = new Person('Jane Doe')
const person2 = new Person('Jane Doe')
person1.getGreetingCallback === person2.getGreetingCallback // true
// and to take it a tiny bit further, these are also both true:
person1.getGreetingCallback === Person.prototype.getGreetingCallback
person2.getGreetingCallback === Person.prototype.getGreetingCallback
```

The nice thing with the module pattern is that it avoids the issues with the callsite we saw above.

```js
const person = getPerson('Jane Doe')
const getGreeting = person.getGreeting
// later...
getGreeting() // Hello Jane Doe
```

We don’t need to concern ourselves with `this` at all in that case. And there are other issues with relying heavily on closures to be aware of. It’s all about trade-offs.

## Private properties with classes

If you really do want to use `class` and have private capabilities of closures, then you may be interested in [this proposal](https://github.com/tc39/proposal-class-fields) (currently stage-2, but unfortunately no babel support yet):

```js

class Person {
  #name
  greeting = 'hey there'
  #getPrefixedName = (prefix) => `${prefix}: ${this.#name}`
  constructor(name) {
    this.#name = name
  }
  setName(strName) {
    #name = strName
    // look at this! shorthand for:
    // this.#name = strName
  }
  getName() {
    return #getPrefixedName('Name')
  }
  getGreetingCallback() {
    const {greeting} = this
    return (subject) => `${this.greeting} ${subject}, I'm ${#name}`
  }
}
const person = new Person('Jane Doe')
person.setName('Sarah Doe')
person.greeting = 'Hello'
person.getName() // Name: Sarah Doe
person.getGreetingCallback()('John') // Hello John, I'm Sarah Doe
person.#name // undefined or error or something... Either way it's totally inaccessible!
person.#getPrefixedName // same as above. Woo! 🎊 🎉

```

So we’ve got the solution to the privacy problem with that proposal. However it doesn’t rid us of the complexities of `this`, so I’ll likely only use this in places where I really need performance gains of `class`.

I should also note that you *can* use a WeakMap to get privacy for classes as well, like I demonstrate in the [WeakMap exercises](https://github.com/kentcdodds/es6-workshop/blob/2a6bf446c95a387c8e87c1398444733618265c8a/exercises-final/13_weakmap.test.js#L10-L23) in the [es6–workshop](https://github.com/kentcdodds/es6-workshop).

## Additional Reading

This article by Tyler McGinnis called “Object Creation in JavaScript” is a terrific read: [https://tylermcginnis.com/object-creation-in-javascript-functional-instantiation-vs-prototypal-instantiation-vs-pseudo-e9287b6bbb32/](https://tylermcginnis.com/object-creation-in-javascript-functional-instantiation-vs-prototypal-instantiation-vs-pseudo-e9287b6bbb32/)

If you want to learn about functional programming, I highly suggest [“The Mostly Adequate Guide to Functional Programming”](https://drboolean.gitbooks.io/mostly-adequate-guide/) by Brian Lonsdorf, and (for those of you with a Frontend Masters subscription) [“Functional-Lite JavaScript”](https://github.com/getify/Functional-Light-JS) by Kyle Simpson.