class: middle
background-image: url(img/title-slide.png)

<br/>
# Как modnaKasta трансформировалась

.meta[Александр Соловьëв, CTO modnaKasta]

???

- Шоппинг-клуб
- Более полугода CTO
- Куча интересных челленджей

---

<img src="img/mk.jpg"/>

- 750 RPS
- 3-6k пользователей на сайте днëм одновременно (по версии GA)
- 120 Gb размер базы
- ~80-100k строк Python'а

???

- Это в пике, 30 сентября, 1 апреля
- На выходных 110-130 RPS
- Кода уже меньше

---

class: center, middle

# Много ли это?

## Достаточно, чтоб умирать от нагрузки

???

- Ну т.е. от качества кода многое зависит
- Сейчас расскажу, как оно было

---

# Как оно было

.center.middle[![old](1-mk-old-arch.png)]

???

- Пітон у шапці та їсть листик, що символізує працелюбність та незакомплексованість тварюки
- Все со всеми
- На самом деле сложнее, в MK - еще ktdb (email, задачи) и баннеры

---

# Метадатапары

.center.middle[<img src="2-mdp.png" style="height: 500px"/>]

???

- Слово-нарицательное
- Обычное решение для хранения KV-данных

---

# И в них у нас хранится...

```sql
=# SELECT k.key, k.id, COUNT(p.id)
-# FROM product_metadatapair p
-# JOIN product_metadatakey k ON p.key_id = k.id
-# GROUP BY k.key, k.id
-# ORDER BY COUNT(p.id);
   key    | id |  count
----------+----+----------
 Объем    | 18 |       26
          |  3 |       27
 Описание | 17 |      176
 Номер    |  2 |      443
 Pазмер   |  1 |     3112
 Размер   | 16 | 15212192
(6 rows)
```

???

- По пустому ключу - пустые значения
- Первая буква первого "размер" - английская п

---

# И наличие...

.center.middle[<img src="3-mdp-stock.png" style="height: 500px"/>]

???

- Я бы ожидал сток как один из ключей в MDP
- А почему m2m?
- В результате, 1 MDP - это 1 реальный товар

---

# И корзина!

.center.middle[<img src="4-mdp-bi.png" style="height: 500px"/>]

???

- 25 млн записей в basket item
- 1 BI - 1 товар, а у нас опять m2m

---

# Problems?

- 22 запроса на 1 размер

--

- 110 запросов на 5 размеров

--

- Это исправили кешированием

--

background-image: url(img/facepalm.png)

- Которое завязано на корзину, поэтому кешируется для каждого юзера отдельно

---

# А как должно быть?

.center.middle[<img src="5-sku.png" style="height: 500px"/>]

???

- Никаких m2m, где не надо

---

# А еще заказы

- Заказ создавался на входе в корзину
- Payment'ы создавались при выборе, но никогда не удалялись
- Пустых неоформленных заказов в базе миллионы
- Корзина не синглтон, найти текущую - проблема

---

# Восстановление пароля!

```diff
commit f2c3eaa1e0747075c005d74094a5391b51498cf6
Author: v.ishchenko <v.ishchenko@modnakasta.ua>
Date:   Thu Feb 5 17:56:09 2015 +0200

    Fix password reset send letter

diff --git a/user/urls.py b/urls.py
@@ -166 +166 @@

-    url(r'^reset/(?P<confirm>[A-Za-z0-9\-]+)/$',
+    url(r'^reset/(?P<confirm>[^/]+)/$',
```

---

# И остальное

- Миграций нет
- Размерные сетки картинками
- Баннерная система на KyotoDB/Lua/Python/JS
- Почтовая очередь на KyotoDB на каждом сервере отдельно

---

background-image: url(img/trollface.png)

# При чëм здесь фронтэнд

--

background-image: ""

# .center.middle.invert[Чëрная<br/>пятница]

---

# Теперь

.center.middle[![new](6-mk-new-arch.png)]

---

# Что там у нас

- React
- ClojureScript
- Datascript
- Javelin

---

# React
## Ну какие собственно альтернативы?

---

# ClojureScript
## worth repeating

- Лисп
- Персистентные структуры данных
- Функциональное программирование
- Компилируется в JS
- Минифицируется в advanced режиме Closure Compiler

---

# Архитектура

.center.middle[<img src="7-frontend-arch.png" style="width: 100%"/>]

---

.center[<img src="img/datascript.png" style="width: 100%"/>]

- Персистентная база данных в памяти
- Язык запросов Datalog
- Формат хранения: триплеты
- Ссылки, джойны, агрегации

---

# Получить из базы баннеры

.big[```clojure
(d/q db

  '[:in $ ?contains-now?
    :find ?e
    :where
      [?e :banner/id]
      [?e :starts-at ?start]
      [?e :finishes-at ?finish]
      [(?contains-now? ?start ?finish)]]

  (fn [start finish]
    (time/within?
      start finish (time/now))))
```]

---

# Пример компонента

```clojure
(rum/defc Banners < rum/reactive
  [id]
  (let [all-banners (rum/react BANNERS)
        banners (get all-banners id)]
    [:div {:class "banners"}
      (for [banner banners]
        [:img {:src banner
               :data-banner-id id}])]))
```

- Компонент `Banners`
- Аргумент - `id`, рисует баннера для этого места
- Реагирует на ячейку `BANNERS`
- Рисует список баннеров

---

# Серверный рендеринг

```clj
(defn handler [req res]
  (.writeHead res 200 {"Content-Type" "text/html"})
  (main/render-to-string
    (.-url req)
    (fn [initial content]
      (.end res (render-template initial content)))))

(def app (.createServer http handler))
(.listen app 6000 "0.0.0.0")
```

- `render-to-string` - функция, которая отрисовывает реакт-приложение в строку
- Полученные для рендеринга данные отдаются на клиент

---

# Весь рендеринг

```clj
(defn render-to-string [url callback]
  (router/set-route url)

  (let [comp (router/Root data/db)]
    ;; Отрендерить 1 раз, чтоб запустить AJAX-запросы
    (js/React.renderToString comp)

    ;; Ждëм завершения запросов
    (add-watch xhr/current-queries ::render
      (fn [_ _ _ value]
        (when (zero? (get-xhr-request-count))
          (callback db (js/React.renderToString comp)))))))
```

- `get-xhr-request-count` возвращает количество запросов "в процессе"

---

# И что, удобно?

- React же :-)
- Live reload без потери состояния
- Live reload есть и на сервере
- Структурное редактирование это счастье (paredit)
- Количество кода меньше, чем на JS
- Шаринг кода между сервером и клиентом 👍
- Tooling вокруг очень крутой

???

- Live reload работает хорошо из-за особенностей CLJS

---

class: center, middle

<style>
h1#-map-answer { margin-left: -7em; }
h1#-some-good-questions- { margin-top: -0.7em;}
</style>

# (map answer
# (some good? questions))
