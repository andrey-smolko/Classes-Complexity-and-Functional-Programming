# Classes, Complexity, and Functional Programming

*When I use classes, when I don’t, what I do instead, and why*

When it comes to applications intended to last, I think we all want to have simple code that’s easier to maintain. Where we often really disagree is how to accomplish that. In this blog post I’m going to talk about how I see functions, objects, and classes fitting into that discussion.

## A class

Let’s take a look at an example of a class implementation to illustrate my point:

```js
class Person {
  constructor(name) {
    // common convention is to prefix properties with `_`
    // if they're not supposed to be used. See the appendix
    // if you want to see an alternative
    this._name = name
    this.greeting = 'Hey there!'
  }
  setName(strName) {
    this._name = strName
  }
  getName() {
    return this._getPrefixedName('Name')
  }
  getGreetingCallback() {
    const {greeting, _name} = this
    return (subject) => `${greeting} ${subject}, I'm ${_name}`
  }
  _getPrefixedName(prefix) {
    return `${prefix}: ${this._name}`
  }
}
const person = new Person('Jane Doe')
person.setName('Sarah Doe')
person.greeting = 'Hello'
person.getName() // Name: John Doe
person.getGreetingCallback()('Jeff') // Hello Jeff, I'm Sarah Doe
```

So we’ve declared a `Person` class with a constructor instantiating a few member properties as well as a couple of methods. With that, if we type out the `person` object in the Chrome console, it looks like this:

![pic1](https://cdn-images-1.medium.com/max/800/1*W_x3AY3JFnw2k7hLQnh2GQ.png)

The real benefit to notice here is that most of the properties for this `person` live on the `prototype` (shown as `__proto__` in the screenshot) rather than the instance of `person`. This is not insignificant because if we had ten thousand instances of `person` they would all be able to share a reference to the same methods rather than having ten thousand copies of those methods everywhere.

What I want to focus on now is how many concepts you have to learn to really understand this code and how much complexity those concepts add to your code.

* Objects: Pretty basic. Definitely entry level stuff here. They don’t add a whole lot of complexity by themselves.
* Functions (and closures): This is also pretty fundamental to the language. Closures do add a bit of complexity to your code (and can cause problems if you’re not careful), but you really can’t make it too far in JavaScript without having to learn these. (Learn more here).
* A function/method’s `this` keyword: Definitely an important concept in JavaScript

> My assertion is that `this` is hard to learn and can add unnecessary complexity to your codebase.