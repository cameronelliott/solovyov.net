class: center, middle

# ClojureScript, FRP, etc

## И про то, как люди тратят свободное время

.meta[Alexander Solovyov, How Far Games]

.footer[Bibliotech, Nov 2013]

---

# О чëм

.meta[Читать название надо было вообще-то]

- Приложения в браузере
- Разные фреймворки
- Разные языки
- Странный способ провести выходные

---

# Приложения в браузере

- Везде работают
- Не нужно париться об инсталляции
- HTML - лучший API для интерфейсов
  .meta[(утверждающие обратно - обманывают вас :))]
- Количество подходов - огромное

---

# Подходы

- jQuery .meta[(привет, прошлое десятилетие)]

--
- MVC (Backbone.js, Angular.js, Ember)

--
- Функциональное реактивное программирование
- И люди продолжают выдумывать

---

# jQuery

## Тут нечего обсуждать, сорри

---

# Backbone.js

- Набор для построения фреймворка
- Очень много работы руками
- Ну очень, очень много
- И немножечко тяжело удерживать всë вместе

---

# Angular.js

- Работает
- Чуточку тормозит
- Код сложный
- API большое
- Концепция противная

---

# Ember

- Идеологически стройный .meta[(свойства и зависимости)]
- Тормозит сильнее .meta[(хотя и не должен)]
- Код очень сложный
- И его очень много

---

# FRP, постулаты

- Действие - отстой
- Декларация - форева
- Иерархия обработчиков данных

---

# Варианты

- Flapjax
- Bacon.js
- RxJS

Общего: API неудобное

---

# React: долой теорию

- Иерархия компонентов
  .meta[(ну не формулы...)]
- render - чистая функция
  .meta[(для определëнного определения)]
- FRP руками инженеров, приближëнно

---

# Как это выглядит

```
List = React.createClass({
  render: function() {
    <ul>{
      this.props.list.map(function(x) {
        return <Item text={x}/>;
      })
    }</ul>
}});

<List list={['a', 'b']}/>
```

---

# На практике

- Работает
- Быстро, сухо, удобно
- В Фейсбуке куча кода
- У меня уже тоже > 5k

---

# Всë супер?

- Язык тот же
- Достаточно ли всë стройно?

---

# JavaScript

- <s>Типы</s>
- <s>Структуры данных</s>
- <s>Зависимости</s>
- <s>Неймспейсы</s>
- <s>Полиморфизм</s>
- <s>Типовые операции</s>

---

# ClojureScript

- Lisp
- Функциональный
- Неизменяемые (persistent) структуры данных
- Регулярный синтаксис
- Неймспейсы
- Удобная стандартная библиотека

---

# Не Си, конечно

- `func(arg1, arg2)`
  - `(func arg1 arg2)`
- `obj.method(arg)`
  - `(.method obj arg)`
- `{key: "value"}`
  - `{:key "value"}`
- `if (a) { b } else { c }`
  - `(if a b c)`

---

class: middle, center

# Хватит разговоров

## Упростит ли это жизнь?

---

# Hoplon

```
(defn my-list [& items]
  (apply ul
    (for [item items]
      (li :class "item" item))))

<my-list>
  <span>first</span>
  <span>second</span>
</my-list>
```

---

# Результат

```
<ul>
  <li class="item"><span>first</span></li>
  <li class="item"><span>second</span></li>
</ul>
```

---

# FRP

```
(defcell clicks 0)

<body>
  <p><text>
      You've clicked ~{clicks} times.
  </text></p>
  <button
     on-click="{{ #(swap! clicks inc) }}">
    Click me
  </button>
</body>
```

---

# Всë супер?

- Скорость
- Она уже не отвратительна, но...

---

class: center, middle

# pump!

### Но начнëм с другой стороны

---

# ClojureCup 28-29.09 '13

.center[![wm](http://warmagnet.com/static/logo/logo-400.png)]

---

.full[![screen](https://api.monosnap.com/image/download?id=eian7diLKLVlmfVIBLDoebwl5)]

---

# Требования

- Достаточная скорость
- Писать прямо сейчас
- Минимально времени на поддержку
- Не скатываться в плохой код
- ClojureScript

---

# pump

Прямолинейная обëртка, лишь бы можно было использовать React без боли

```
(defr MyList [{:keys [list]}]
  [:ul
    (for [item items]
      [:li {:class "item"} item])])

<MyList list={["a", "b"]}/>
```

---

# Потенциальные проблемы

- Скорость
- Непривычность
- Размеры результатов

---

# Скорость

- Проблемы не заметили
- ;)

---

# Непривычность

- Поначалу слишком плотно
- Без paredit'а - тяжело редактировать
- Неизменяемость данных поначалу тяжело ложится на сознание
- Вход тем не менее был довольно быстрый
  - За полдня субботы все начали писать более-менее
  - Как из Ланоса в Мондео (назад - не очень)

---

# Плотность

- 900 loc CLJS
- ~1500-3000 loc Coffee по прикидкам разных людей
  - Спекуляции спекуляциями, а компоненты сильно короче, чем на JS
- Через месяц код опять слишком плотный
  - Требуется время, чтоб вникнуть

---

# Размеры

- CLJS **877** loc, Clojure **515** loc, общего **111** loc

--
- React **430kb**, WarMagnet **1.2mb**

--
- Min: React **57kb** (**21kb** gzip), WarMagnet **300kb** (**68kb** gzip)

--
- Для сравнения, jQuery - **33kb** gzip

---

# Архитектура

- Event sourcing
- Append log событий
- Функция модификации состояния игры общая между клиентом и сервером


---

# Aftermath

- Paredit'а сильно не хватает в остальных языках™
- Работать с данными в остальных языках™ неудобно
- От изменяемых данных побаливают зубы и чувство опасности не покидает
- Теперь я страдаю на работе!
  .meta[(тока никому ни слова! :)]

---

class: center, middle

# Это почти всë

## Остались только вопросы :)

---

# Ссылки

- http://github.com/wunderwaffle/warmagnet/
- http://github.com/piranha/pump/
- http://facebook.github.io/react/
- http://piranha.github.io/slides/
- https://github.com/tailrecursion/hoplon
