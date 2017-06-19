# How to avoid `this`

So, if `this` adds so much complexity (as I’m asserting), how do we avoid it without adding even more complexity to our code? How about instead of the object-oriented approach of classes, we try a more functional approach? This is how things would look if we used [pure functions](https://en.wikipedia.org/wiki/Pure_function):

```js
function setName(person, strName) {
  return Object.assign({}, person, {name: strName})
}

// bonus function!
function setGreeting(person, newGreeting) {
  return Object.assign({}, person, {greeting: newGreeting})
}

function getName(person) {
  return getPrefixedName('Name', person.name)
}

function getPrefixedName(prefix, name) {
  return `${prefix}: ${name}`
}

function getGreetingCallback(person) {
  const {greeting, name} = person
  return (subject) => `${greeting} ${subject}, I'm ${name}`
}

const person = {greeting: 'Hey there!', name: 'Jane Doe'}
const person2 = setName(person, 'Sarah Doe')
const person3 = setGreeting(person2, 'Hello')
getName(person3) // Name: Sarah Doe
getGreetingCallback(person3)('Jeff') // Hello Jeff, I'm Sarah Doe
```

With `this` solution we have no reference to this. We don’t have to think about it. As a result, it’s easier to understand. Just functions and objects. There is basically no state you need to keep in your head at all with these functions which makes it very nice! And the person object is just data, so even easier to think about:

![pic1](https://cdn-images-1.medium.com/max/800/1*dWZiYJoerNxSseHZ_XfKPA.png)

Another nice property of functional programming that I won’t delve into very far is that it’s very easy to unit test. You simply call a function with some input and assert on its output. You don’t need to set up any state beforehand. That’s a very handy property!

Note that functional programming is more about making code easier to understand so long as it’s “fast enough.” Despite speed of execution not being the focus, there are some reeeeally nice perf wins you *can* get in certain scenarios (like reliable `===` equality checks for objects for example). More often than not, **your use of functional programming will** *often be way down on the list of bottlenecks that are making your application slow.*

## Cost and Benefit

Usage of `class` is not bad. It definitely has its place. If you have some really [“hot” code](https://en.wikipedia.org/wiki/Hot_spot_%28computer_programming%29) that’s a bottleneck for your application, then using `class` can really speed things up. But 99% of the time, that’s not the case. And I don’t see how `classes` and the added complexity of `this` is worth it for most cases (let’s not even get started with [prototypal inheritance](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)). I have yet to have a situation where I needed `classes` for performance. So I only use them for React components because that’s what you have to do if you need to use state/lifecycle methods (but maybe not in the [future](https://github.com/reactjs/react-future/tree/master/07%20-%20Returning%20State)).

## Conclusion

Classes (and prototypes) have their place in JavaScript. But they’re an optimization. They don’t make your code simpler, they make it more complex. It’s better to narrow your focus on things that are not only simple to learn but simple to understand: functions and objects.