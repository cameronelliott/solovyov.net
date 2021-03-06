class: center, middle

# React

## или как начать жить проще

.meta[Alexander Solovyov, How Far Games]

.footer[Frontend DevConf '14]

---

# Начнëм с начала

## Как писать большие программы

- Разделение ответственностей
- Повышение связности .transparent[(cohesion)]
- Уменьшение связанности .transparent[(coupling)]

---

# Связанность

Степень, в которой каждый программный модуль полагается на другие модули.

> Знает ли `A` про `B`?

---

# Связность

Степень внутренней взаимосвязи между частями одного модуля.

> Есть ли смысловая связь между `A.fn1` и `A.fn2`?

---

# HTML vs JS

- Отображение vs Логика отображения
- Как показывать vs Что показывать
- Связанность - высокая
- И связность - тоже!

---

# Нас всю жизнь обманывали!

- Нужно решать самому, у кого какая ответственность
- Логика отображения неотрывна от его описания

---

class: center, middle

# Шаблоны — это разделение .highlight[технологий], а не ответственностей.

---

# Компоненты

- Иерархические
- Высокая связность
- Простой протокол, поощряющий низкую связанность


- Легко переиспользовать
- Легко компоновать

---

# Пример

```javascript
var Text = React.createClass({
  render: function() {
    return React.DOM.p(
        nil, this.props.text);
  }
});

var Root = React.createClass({
  render: function() {
    return Text({text: "hello"});
  }
});
```

---

# Как же верстальщики?


```javascript
var Text = React.createClass({
  render: function() {
    return <p>{this.props.text}</p>
  }
});

var Root = React.createClass({
  render: function() {
    return <Text text="hello"/>;
  }
});
```

---

# Как насчëт спагетти

- Пишите хорошо!
- Не мешайте бизнес-логику и логику отображения
- Компоненты вверху иерархии больше контроллеры

---

# DOM

- Отложенный (retained) режим
- На каждое изменение — проверь исходную позицию
- Комбинаций всех вариантов — **очень** много

---

# Непосредственный режим

- Декларативное описание
- Полная перерисовка при изменении данных
- Viva la revolución

---

# <s>This is madness</s>

- `React.DOM.*` возвращают т.н. Virtual DOM
- diff(VDOM<sub>n-1</sub>, VDOM<sub>n</sub>) -> DOM
- **Быстро**

---

# Как

- Вычисляет минимальные необходимые изменения
- Применяет пачкой
- Автоматическая делегация событий
- 60 fps в UIWebView на iPhone

---

# Победа?

- Компоненты — чистые функции
- Входящие данные → `render` → DOM
- Отсутствие магии

---

# Еще плюсы?

- Рендеринг на сервере (Node.js, JVM Nashorn)
- Синтетические события (ближе к стандартам, чем любой браузер)
- Хуки для связи с другими библиотеками
- Поддержка SVG
- Отличный фундамент для чего угодно

---

# Ссылки

- http://reactjs.org/
- http://facebook.github.io/react/docs/videos.html
- http://solovyov.net/
