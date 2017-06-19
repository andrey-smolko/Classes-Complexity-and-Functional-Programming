# Ключевое слово `this`

Вот что написано в [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this) по поводу `this`:

> Поведение **ключевого слова `this`** в JavaScript несколько отличается по сравнению с остальными языками. Имеются также различия при использовании `this` в [строгом](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) и нестрогом режиме.
В большинстве случаев значение `this` определяется тем, каким образом вызвана функция. Значение `this` не может быть установлено путем присваивания во время исполнения кода и может иметь разное значение при каждом вызове функции. В ES5 представлен метод [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind), чтобы определить значение ключевого слова this независимо от того, как вызвана функция. Также в ECMAScript 2015 представлены [стрелочные функции](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions), `this` которых привязан к окружению, в котором была создана стрелочная функция. 

Может быть это и не очень хитрая наука, но намного сложнее, чем объекты и замыкания. Вы не можете избавиться от объектов и замыканий, но я уверен что вы в большинстве случаев можете обойтись без использования классов.

Вот (придуманный) пример кода, в котором неправильное использование `this` приводит к непредсказуемым результам. (прим. данный фрагмент написан в строгом режиме)

```js
const person = new Person('Jane Doe')
const getGreeting = person.getGreeting
// далее...
getGreeting() // Uncaught TypeError: Cannot read property 'greeting' of undefined at getGreeting
```

>Проблема заключается в том, что ваша функция зависит от способа вызова, так как содержит `this`.

В качестве примера рассмотрим более реальный случай, который особо применим к React ⚛️. Если вы хоть раз использовали React, у вас возможно возникала таже ошибка, что и у меня:

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

Когда вы нажмете на кнопку вы увидите ошибку: `Uncaught TypeError: Cannot read property 'setState' of null at increment`

И все это из-за использования `this`, потому что мы передаем наш метод в `onClick`, который вызывает метод `increment` с `this` который не ссылается на экземпляр нашего компонента. Существует несколько способов исправить эту ошибку ([посмотрите бесплатные 🆓 видео 💻 на egghead.io](https://egghead.io/lessons/javascript-public-class-fields-with-react-components)).

> Тот факт, что вам необходимо думать о правильном использовании `this` дает дополнительную нагрузку на чтение и понимании вашего кода. А это именно то, от чего мы хотим избавиться.