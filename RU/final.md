# Классы, Сложность и Функциональное Программирование

Перевод статья от Kent C. Dodds [Classes, Complexity, and Functional Programming](https://medium.com/@kentcdodds/classes-complexity-and-functional-programming-a8dd86903747)

*Когда я использую классы, когда не использую, и что я использую вместо них и почему*

Когда наше приложение начинает свою жизнь, я думаю, что мы все хотим, чтобы внутри был простой код, который легко поддерживать. Но в чем мы часто не можем прийти к общему пониманию, так это в том, как достичь простоты. В этой статье я собираюсь рассказать о своем понимании функций, объектов и классов в вышеупомянутом контексте.

## Класс

Чтобы проиллюстрировать мою точку зрения, давайте посмотрим на примере реализации класса:

```js
class Person {
  constructor(name) {
    // Принято к скрытым от использования извне свойствам 
    // добавлять префикс '_'. В приложении
    // можно будет увидеть другие варианты.
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

Итак, мы описали класс `Person` в конструкторе которого определяется пара свойств, а также создает несколько методов. Теперь, если мы в консоли Chrome напишем `person` (созданный экземпляр класс `Person`), то увидим вот что:

![pic1](https://cdn-images-1.medium.com/max/800/1*W_x3AY3JFnw2k7hLQnh2GQ.png)

Настоящая выгода, которую мы можем заметить, заключается в том, что большинсво свойств объекта `person` находится в прототипе `prototype` (см. `__proto__` на скриншоте), а не в самом экземпляре объекта. Это достаточно важно, потому что если у нас будет 10 000 экземпляров `person`, то все они будет иметь доступ к одному и тому же прототипу, а не создавать 10 000 копий реализации этих свойств.

На что хотелось бы обратить внимание, так это на то сколько разных концепций необходимо изучить, чтобы понять полностью понять этот код, и сколько сложностей эти концепции привносят в ваш код.

* Объекты: Довольно просто. Определенно материал начального уровня. Они не привносят много сложностей сами по себе.
* Функции (и [замыкания](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)): Это тоже очень фундаментальные основы языка. Замыкания действительно добавляют некоторую сложность в ваш код (и могут вызвать [проблемы](https://blog.meteor.com/an-interesting-kind-of-javascript-memory-leak-8b47d2e7f156), если вы используйте их неосторожно), но вы не сможете нормально программировать в Javascript, если не изучите замыкания.
* Ключевое слово `this` в функциях и методах: Определенно важное понятие в Javascript.

> Я утверждаю то, что `this` тяжело понять и оно может добавить ненужную сложность в ваш код.

## Ключевое слово `this`

Вот что написано в [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this) по поводу `this`:

> Поведение **ключевого слова `this`** в JavaScript несколько отличается по сравнению с остальными языками. Имеются также различия при использовании `this` в [строгом](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) и нестрогом режиме.
В большинстве случаев значение `this` определяется тем, каким образом вызвана функция. Значение `this` не может быть установлено путем присваивания во время исполнения кода и может иметь разное значение при каждом вызове функции. В ES5 представлен метод [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind), чтобы определить значение ключевого слова this независимо от того, как вызвана функция. Также в ECMAScript 2015 представлены [стрелочные функции](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions), `this` которых привязан к окружению, в котором была создана стрелочная функция.

Может быть это и не очень хитрая наука, но намного сложнее, чем объекты и замыкания. Вы не можете избавиться от объектов и замыканий, но я уверен, что вы в большинстве случаев можете обойтись без использования классов и `this`.

Вот (придуманный) пример кода, в котором неправильное использование `this` приводит к непредсказуемым результатам. (*прим. Webpurple* данный фрагмент выполняется в строгом режиме)

```js
const person = new Person('Jane Doe')
const getGreeting = person.getGreeting
// далее...
getGreeting() // Uncaught TypeError: Cannot read property 'greeting' of undefined at getGreeting
```

> Ключевая проблема заключается в том, что на вашу функцию влияет способ вызовы, так как содержит `this`.

В качестве примера рассмотрим более реальный случай, который относится к React ⚛️. Если вы хоть раз использовали React, у вас, возможно, возникала та же ошибка, что и у меня:

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

И все это из-за использования `this`, потому что мы передаем наш метод в `onClick`, который вызывает метод `increment` с `this` который не ссылается на экземпляр нашего компонента. Существует несколько способов обойти эту ошибку ([посмотрите бесплатные 🆓 видео 💻 на egghead.io](https://egghead.io/lessons/javascript-public-class-fields-with-react-components)).

> Тот факт, что вам необходимо думать о правильном использовании `this` дает дополнительную когнитивную нагрузку. А это именно то, от чего было бы хорошо избавиться.

## Как избежать `this`

Итак, если `this` добавляет столько сложности (как я утверждаю), как же нам избавиться от него, при этом не добавив еще больше сложности в код? Как насчет того чтобы вместо классического объектно-ориентированного подхода, мы попробуем более функциональный подход? Вот что получится, если мы будем использовать [чистые функции](https://en.wikipedia.org/wiki/Pure_function):

```js
function setName(person, strName) {
  return Object.assign({}, person, {name: strName})
}

// Бонусная функция!
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

В этом случае у нас нет никакой связи с `this`. Нам не надо думать об этом. И, как результат, это намного легче понять. Только функции и объекты. Нет ничего дополнительного, что вам необходимо держать в голове. `person` это всего лишь объект с данными. Вот что показывает консоль в Chrome:

![pic1](https://cdn-images-1.medium.com/max/800/1*dWZiYJoerNxSseHZ_XfKPA.png)

В функциональном программировании есть еще один приятный бонус, в который мы не будем сильно вдаваться - эти функции очень легко тестировать. Вы просто вызываете функцию с неким набором параметров и сравниваете с выходным значение. На мой взгляд, это очень приятный бонус!

Функциональное программирование более сосредоточенно на том, чтобы сделать код проще в понимании, чем на увеличении скорости работы. Несмотря на то что скорость не является приоритетом, в некоторых ситуациях вы *можем* использовать ряд оптимизаций (например использование строгого `===` для сравнивания объектов). В большинстве случаев, **ваше использование функционального программирования** *будет в конце списка узких мест, которые замедляют ваше приложение.*

## Недостатки и Преимущества

Использование `class` - это не плохо. У него действительно есть свое место. Если у вас есть [Хот-спот](https://ru.wikipedia.org/wiki/%D0%A5%D0%BE%D1%82-%D1%81%D0%BF%D0%BE%D1%82_(%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5)) который является узким местом вашего приложения, тогда использования `class` может реально повысить производительность. Но в 99% случаях это не так актуально. Поэтому я не вижу, как `классы` и сложность использования `this` стоят потраченных усилий на написание и поддержку кода (и мы даже не начинали говорить [прототипном наследовании](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)). У меня пока еще не было такого опыта в разработке, когда мне действительно были нужны `классы` для оптимизации производительности. Так что я использую их только для компонентов React, потому что это единственное что нам остается, если мы хотим использовать state и lifecycle методы (но возможно в [скором времени это изменится](https://github.com/reactjs/react-future/tree/master/07%20-%20Returning%20State)).

## Заключение

У классов (и прототипов) есть свое место в Javascript. Но оно, как правило, касается оптимизации. Они не делают ваш код проще, а наоборот усложняют. Лучше всего сфокусироваться на вещах, которые не только проще в изучении, но и проще в понимании: функции и объекты.

## Приложение

Здесь находятся некоторые дополнения к статье. Смотрите, читайте и получайте удовольствие :)

### Паттерн Модуль

Паттерн Модуль - это другой путь избежать сложности, которую несет `this` и использовать простые объекты и функции. Больше про этот паттерн можно узнать из книги Эдди Османи (Addy Osmani) [“Learning JavaScript Design Patterns”](https://addyosmani.com/resources/essentialjsdesignpatterns/book/), которая бесплатно для чтения доступна [здесь](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript). Ниже представлена реализация нашего класса `person` используя паттерн Открытый Модуль (“Revealing Module Pattern”):

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

Мне нравится в этом подходе то, что его легко понять. Все просто - у нас есть функция, которая создает несколько переменных и возвращает объект. Просто объекты и функции. Для справки, на рисунке ниже представлен полученный объект `person`, если на него посмотреть в Chrome:

![pic1](https://cdn-images-1.medium.com/max/800/1*qnMDnUmvNaMRuU2ic9ctOQ.png)

Один из недостатков паттерна Модуль состоит в том, что каждый экземпляр  `person` имеет собственные копии всех свойств и методов. Например:

```js
const person1 = getPerson('Jane Doe')
const person2 = getPerson('Jane Doe')
person1.getGreetingCallback === person2.getGreetingCallback // false
```

Несмотря на то, что содержимое функции `getGreetingCallback` идентично в обоих случаях, они будут храниться в памяти как два разных объекта. В большинстве случаев это не является проблемой, но, если вы планируете создавать тонны экземпляров, или вы хотите создавать экземпляры ну просто очень быстро, тогда это может стать проблемой. В нашем предыдущем примере с классом `Person` каждый экземпляр, который мы создаем будет содержать ссылку на один и тот же метод `getGreetingCallback`:

```js
const person1 = new Person('Jane Doe')
const person2 = new Person('Jane Doe')
person1.getGreetingCallback === person2.getGreetingCallback // true

person1.getGreetingCallback === Person.prototype.getGreetingCallback // true
person2.getGreetingCallback === Person.prototype.getGreetingCallback // true
```

Одно из достоинств паттерна Модуль состоит в том, что он позволяет избежать проблем, вызванных различным поведением функции в зависимости от места ее вызова. Одну из таких проблем мы уже встречали выше.

```js
const person = getPerson('Jane Doe')
const getGreeting = person.getGreeting
// later...
getGreeting() // Hello Jane Doe
```

В таком случае нам больше не нужно беспокоиться о `this`. Но при таком подходе стоит знать о [других проблемах](https://twitter.com/BrendanEich/status/871876967796056067), которые могут появиться, если активно полагаться на замыкания. Везде есть компромисс.

### Приватные свойства в классах

Если вы действительно хотите использовать `классы` и при этом иметь возможность создавать приватные поля, которую дают замыкания, тогда вам может быть интересно [это предложение](https://github.com/tc39/proposal-class-fields) (в настоящее время находится в стадии stage-2, но, к сожалению, на момент написания статьи еще не поддерживается babel).

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
    // смотрите! сокращенная запись для:
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
person.#name // undefined or error or something... В любом случае свойство полностью недоступно!
person.#getPrefixedName // аналогично предыдущей строке. Вау!🎊🎉
```

Итак, у нас есть решения проблемы приватности с помощью данного предложения. Однако, это все-равно не избавляет нас от сложности использования `this`, поэтому скорее всего я бы использовал такой подход только там, где действительно нужен прирост производительности, который можно получить, используя `класс`.

Должен отметить, что вы также *можете* использовать WeakMap для реализации приватности в классах, например, смотри мой [WeakMap exercises](https://github.com/kentcdodds/es6-workshop/blob/2a6bf446c95a387c8e87c1398444733618265c8a/exercises-final/13_weakmap.test.js#L10-L23), который входит в [es6–workshop](https://github.com/kentcdodds/es6-workshop).

### Что почитать дополнительно

Великолепная статься от Tyler McGinnis, которая называется “Object Creation in JavaScript” [https://tylermcginnis.com/object-creation-in-javascript-functional-instantiation-vs-prototypal-instantiation-vs-pseudo-e9287b6bbb32/](https://tylermcginnis.com/object-creation-in-javascript-functional-instantiation-vs-prototypal-instantiation-vs-pseudo-e9287b6bbb32/)

Если вы хотите больше узнать о функциональном программировании, я настоятельно рекомендую книгу от Brian Lonsdorf [“The Mostly Adequate Guide to Functional Programming”](https://drboolean.gitbooks.io/mostly-adequate-guide/) и книгу от Kyle Simpson [“Functional-Lite JavaScript”](https://github.com/getify/Functional-Light-JS)