# Экземпляры приложения & компонента

## Создание экземпляра приложения

Каждое приложение начинается с создания нового **экземпляра приложения** с помощью функции `createApp`:

```js
const app = Vue.createApp({ /* опции */ })
```

Экземпляр приложения используется для регистрации глобальных вещей, которые затем могут использоваться компонентами внутри этого приложения. Подробнее об этом мы обсудим дальше в руководстве, но в качестве небольшого примера:

```js
const app = Vue.createApp({})
app.component('SearchInput', SearchInputComponent)
app.directive('focus', FocusDirective)
app.use(LocalePlugin)
```

Большинство методов, предоставляемых экземпляром приложения, возвращают этот же экземпляр, что позволяет собирать вызовы в цепочки:

```js
Vue.createApp({})
  .component('SearchInput', SearchInputComponent)
  .directive('focus', FocusDirective)
  .use(LocalePlugin)
```

Посмотреть полный список API приложения можно в разделе [Справочник API](../api/application-api.md).

## Корневой компонент

Опции, передаваемые в `createApp`, используются для настройки **корневого компонента**. Он используется в качестве стартовой точки для отрисовки, при **монтировании** приложения.

Приложение должно быть примонтировано в DOM-элемент. Например, если требуется примонтировать приложение Vue в `<div id="app"></div>`, то необходимо передать `#app`:

```js
const RootComponent = { /* опции */ }
const app = Vue.createApp(RootComponent)
const vm = app.mount('#app')
```

В отличие от большинства методов приложения `mount` не возвращает экземпляр приложения, вместо этого он возвращает экземпляр корневого компонента.

Пусть Vue и не реализует [паттерн MVVM](https://ru.wikipedia.org/wiki/Model-View-ViewModel) в полной мере, архитектура фреймворка во многом вдохновлена им. Поэтому переменную с экземпляром приложения традиционно именуют `vm` (сокращённо от ViewModel).

Хоть все примеры на этой странице нуждаются лишь в одном компоненте, в большинстве реальных приложений организовано дерево вложенных, многократно используемых компонентов. Например, дерево компонентов приложения Todo-списка может быть таким:

```
Корневой компонент
└─ TodoList
   ├─ TodoItem
   │  ├─ DeleteTodoButton
   │  └─ EditTodoButton
   └─ TodoListFooter
      ├─ ClearTodosButton
      └─ TodoListStatistics
```

Каждый компонент будет иметь свой собственный экземпляр компонента `vm`. У некоторых компонентов, таких как `TodoItem`, вероятно будет несколько экземпляров, отображаемых в один момент времени. Все экземпляры компонентов в этом приложении будут иметь один и тот же экземпляр приложения.

На [системе компонентов](component-basics.md) подробнее остановимся позже. Сейчас достаточно запомнить, что корневой компонент на самом деле ничем не отличается от любого другого компонента. Опции конфигурации такие же, как и поведение соответствующего экземпляра компонента.

## Свойства экземпляра компонента

Ранее в руководстве встречались свойства `data`. Доступ к свойствам, определённым в `data`, предоставляется через экземпляр компонента:

```js
const app = Vue.createApp({
  data() {
    return { count: 4 }
  }
})

const vm = app.mount('#app')

console.log(vm.count) // => 4
```

Существуют и другие опции компонента, которые добавляют определяемые пользователем свойства к экземпляру компонента, например `methods`, `props`, `computed`, `inject` и `setup`. Подробнее каждую из этих опций изучим дальше в руководстве. Все свойства экземпляра компонента, независимо от того, как они определены, будут доступны в шаблоне компонента.

Vue также предоставляет доступ к некоторым встроенным свойствам через экземпляр компонента, например `$attrs` и `$emit`. У таких свойств всегда будет префикс `$` во избежание конфликта имён со свойствами, определёнными пользователем.

## Хуки жизненного цикла

Каждый экземпляр при создании проходит через последовательность шагов инициализации — например, настраивает наблюдение за данными, выполняет компиляцию шаблона, монтирует экземпляр в DOM, обновляет DOM при изменении данных. Между этими шагами вызываются функции, называемые **хуками жизненного цикла**, с помощью которых можно выполнять свой код на определённых этапах.

Например, хук [`created`](../api/options-lifecycle-hooks.md#created) можно использовать для выполнения кода после создания экземпляра:

```js
Vue.createApp({
  data() {
    return { count: 1 }
  },
  created() {
    // `this` указывает на экземпляр vm
    console.log('счётчик: ' + this.count) // => "счётчик: 1"
  }
})
```

Также есть и другие хуки, которые будут вызываться на различных этапах жизненного цикла экземпляра, например [`mounted`](../api/options-lifecycle-hooks.md#mounted), [`updated`](../api/options-lifecycle-hooks.md#updated) и [`unmounted`](../api/options-lifecycle-hooks.md#unmounted). Все хуки вызываются с контекстом `this`, указывающим на текущий активный экземпляр, который их вызвал.

:::tip Совет
Не используйте [стрелочные функции](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Functions/Arrow_functions) в свойствах экземпляра и в коллбэках, например `created: () => console.log(this.a)` или `vm.$watch('a', newVal => this.myMethod())`. Так как стрелочные функции не имеют собственного `this`, то `this` в коде будет обрабатываться как любая другая переменная и её поиск будет производиться в областях видимости выше до тех пор пока не будет найдена, часто приводя к таким ошибкам, как `Uncaught TypeError: Cannot read property of undefined` или `Uncaught TypeError: this.myMethod is not a function`.
:::

## Диаграмма жизненного цикла

Ниже представлена диаграмма жизненного цикла экземпляра. Необязательно понимать её полностью прямо сейчас, но по мере изучения и практики разработки к ней полезно будет обращаться.

<img src="/images/lifecycle.png" width="840" height="auto" style="margin: 0px auto; display: block; max-width: 100%;" loading="lazy" alt="Хуки жизненного цикла экземпляра">
