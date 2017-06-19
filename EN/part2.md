# The `this` keyword

Here’s what [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this) has to say about `this`:

> A **function’s `this` keyword** behaves a little differently in JavaScript compared to other languages. It also has some differences between [strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) and non-strict mode.
In most cases, the value of `this` is determined by how a function is called. It can't be set by assignment during execution, and it may be different each time the function is called. ES5 introduced the [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) method to set the value of a function's `this` regardless of how it's called, and ES2015 introduced [arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) whose `this` is lexically scoped (it is set to the this value of the enclosing execution context).

Maybe not rocket science 🚀, but definitely more complicated than just objects and closures. You can’t get away from objects and closures, but I believe you can often get away with avoiding classes and this most of the time.

Here’s a (contrived) example of where things can break down with `this`.

```js
const person = new Person('Jane Doe')
const getGreeting = person.getGreeting
// later...
getGreeting() // Uncaught TypeError: Cannot read property 'greeting' of undefined at getGreeting
```

>The core issue is that your function has been “complected” with wherever it is referenced because it uses `this`.

For a more real world example of the problem, you’ll find that this is especially evident in React ⚛️. If you’ve used React for a while, you’ve probably made this mistake before as I have:

```js
class Counter extends React.Component {
  state = {clicks: 0}
  increment() {
    this.setState({click: this.state.clicks++})
  }
  render() {
    return (
      <button onClick={this.increment}>
        You have clicked me {this.state.clicks} times
      </button>
    )
  }
}
```

When you click the button you’ll see: `Uncaught TypeError: Cannot read property 'setState' of null at increment`

And this is all because of `this`, because we’re passing it to `onClick` which is not calling our `increment` function with `this` bound to our instance of the component. There are various ways to fix this ([watch this free 🆓 egghead.io video 💻 about how](https://egghead.io/lessons/javascript-public-class-fields-with-react-components)).

> The fact that you have to think about `this` adds cognitive load that would be nice to avoid.