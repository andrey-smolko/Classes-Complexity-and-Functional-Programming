# Приложение

Здесь находится некоторые дополнения к статье, смотрите с удовольствием :)

## Паттерн Модуль

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

Мне нравится в этом подходе то, что его легко понять. Все просто - у нас есть функция, которая создает несколько локальных переменных и возвращает объект. Просто объекты и функции. Для справки, на рисунке ниже представлен полученный объект `person`, если на него посмотреть в Chrome DevTools:

![pic1](https://cdn-images-1.medium.com/max/800/1*qnMDnUmvNaMRuU2ic9ctOQ.png)

Один из недостатков паттерна Модуль состоит в том, что каждый экземляр `person` имеет собственные копии всех свойств и методов. Например:

```js
const person1 = getPerson('Jane Doe')
const person2 = getPerson('Jane Doe')
person1.getGreetingCallback === person2.getGreetingCallback // false
```

Несмотря на то, что содержмое функции `getGreetingCallback` идентично в обоих случаях, они будут храниться в памяти как два разных объекта. В большинстве случаев это не является проблемой, но если вы планируете создавать тонны экземпляров или вы хотите создавать экземпляры ну просто очень быстро, тогда это может стать проблемой. В нашем предыдущем примере с классом `Person` каждый экземпляр, который мы создаем будет содержать ссылку на один и тот же метод `getGreetingCallback`:

```js
const person1 = new Person('Jane Doe')
const person2 = new Person('Jane Doe')
person1.getGreetingCallback === person2.getGreetingCallback // true

person1.getGreetingCallback === Person.prototype.getGreetingCallback // true
person2.getGreetingCallback === Person.prototype.getGreetingCallback // true
```

Одно из достоинств паттерна Модуль состоит в том, что он позволяет избежать проблем вызванных различным поведением функции в зависимости от места ее вызова. Одну из таких проблем мы уже встречали выше.

```js
const person = getPerson('Jane Doe')
const getGreeting = person.getGreeting
// later...
getGreeting() // Hello Jane Doe
```

В таком случае нам больше не нужно беспокоиться о `this`. Но при таком подходе стоит знать о [других проблемах](https://twitter.com/BrendanEich/status/871876967796056067), которые могут появиться, если широко полагаться на замыкания. Везде есть компромисс.

## Приватные свойсва в классах

Если вы действительно хотите использовать `классы` и при этом иметь возможность создавать приватные поля, которую дают замыкания, тогда вам может быть интересно [это предложение](https://github.com/tc39/proposal-class-fields) (в настоящее время находится в стадии-2, но к сожалению на момент написания статьи еще не поддерживается babel).

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
    // смотрите! сокращение для:
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
person.#getPrefixedName // аналогично предыдущей строке. Woo! 🎊 🎉
```

Итак, у нас есть решения проблемы приватности с помощью данного предложения.