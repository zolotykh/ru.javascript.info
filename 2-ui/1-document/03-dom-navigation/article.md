libs:
  - d3
  - domtree

---


# Хождение по DOM

DOM позволяет нам делать различные операции с элементами и их содержимым. Но для начала нам необходимо получить соответствующий DOM объект.

Все операции с DOM начинаются с объекта `document`. Используя его мы можем получить доступ к любому узлу.

На изображении ниже представлены связи, используя которые можно перемещаться между DOM узлами.

![](dom-links.png)

Поговорим о них более подробно.

## На самом верху: documentElement и body

Самые верхние 3 узла доступны напрямую как свойства `document`.

`<html>` = `document.documentElement`
: Самый верхний узел документа - `document.documentElement`. Этот DOM узел представляет `<html>` тег.

`<body>` = `document.body`
: Другой широко используемый DOM узел это `<body>` элемент -- `document.body`.

`<head>` = `document.head`
: Тег `<head>` доступен как `document.head`.

````warn header="Возможна проблема: `document.body` может быть `null`"
Скрипт не может получить доступ к элементу, который не существует в момент выполнения кода.

Особенно, если скрипт расположен внутри тега `<head>`, тогда `document.body` не доступен, т.к. браузер ещё не обработал его.

Таким образом, в примере ниже первый `alert` показывает `null`:

```html run
<html>

<head>
  <script>
*!*
    alert( "Из HEAD: " + document.body ); // null, Тега <body> ещё нет
*/!*
  </script>
</head>

<body>

  <script>
    alert( "Из BODY: " + document.body ); // HTMLBodyElement, теперь существует
  </script>

</body>
</html>
```
````

```smart header="В мире DOM `null` означает \"не существует\""
В DOM, значение `null` означает "не существует" или "нет такого узоа".
```

## Дочерние узлы: childNodes, firstChild, lastChild

Два термина, которые мы будем теперь использовать:

- **Дочерние узлы (или children)** -- элементы, которые находятся следующими по уровню вложенности. Иными словами, они вложены прямо в данный узел. Для примера, `<head>` и `<body>` дочерние узлы `<html>` элемента.
- **Потомки** -- все элементы, которые вложены в текущий, и могущие также содержать потомков.

Для примера, здесь `<body>` имеет дочерние `<div>` и `<ul>` теги (и несколько пустых текстовых нод):

```html run
<html>
<body>
  <div>Начало</div>

  <ul>
    <li>
      <b>Информация</b>
    </li>
  </ul>
</body>
</html>
```

...И все потомки элемента `<body>` не только первые `<div>` и `<ul>` в уровне вложенности, но также и все остальные вложенные узлы, такие как `<li>` (дочерний узел от `<ul>`) и `<b>` (дочерний узел от `<li>`) -- целое поддерево.

**Свойство `childNodes` это коллекция предоставляющая доступ ко все вложенным узлам, включая текстовые.**

Пример ниже демонстрирует дочерние узлы от `document.body`:

```html run
<html>
<body>
  <div>Начало</div>

  <ul>
    <li>Информация</li>
  </ul>

  <div>Конец</div>

  <script>
*!*
    for (let i = 0; i < document.body.childNodes.length; i++) {
      alert( document.body.childNodes[i] ); // Text, DIV, Text, UL, ..., SCRIPT
    }
*/!*
  </script>
  ...ещё какая-то разметка...
</body>
</html>
```

Пожалуйста обратите внимание на интересную деталь. Если мы запустим пример выше, последний показанный элмент - `<script>`. По факту, документ может содержать больше узлов ниже, но в момент выполнения этого кода браузер ещё не обработал оставшуюся часть документа.

**Свойства `firstChild` и `lastChild` предоставляют простой способ как обратиться к первому и последнему дочернему узлу родительского элемента.**

Они являются синонимами. Если у узла есть дочерние элементы, то строки ниже всегда вернут true:
```js
elem.childNodes[0] === elem.firstChild
elem.childNodes[elem.childNodes.length - 1] === elem.lastChild
```

Также есть специальная функция `elem.hasChildNodes()`  для проверки наличия дочерних элементов.

### DOM коллекции

Как мы можем видеть, `childNodes` выглядит подобно массиву. Но на самом деле это не массив, а скорее *collection* -- специальный массиво-подобный перечисляемый объект.

Два важных заключения:

1. Мы можем использовать `for..of` для перебора коллекции:
  ```js
  for (let node of document.body.childNodes) {
    alert(node); // показывает все ноды коллекции
  }
  ```
  Все потому что он перечисляемый (предоставляет свойство `Symbol.iterator`, как обязательное).

2. Методы массивы не будут работать, потому что это не массив:
  ```js run
  alert(document.body.childNodes.filter); // undefined (метода filter не существует!)
  ```

Первое заключение хорошо. Второе терпимо, потому что мы можем использовать `Array.from` для создания "настоящего" массива из коллекции, если нам нужны методы массива:

  ```js run
  alert( Array.from(document.body.childNodes).filter ); // теперь методы массива существуют
  ```

```warn header="DOM коллекции доступны только для чтения"
DOM коллекции, и даже больше -- *все* свойства навигации показанные в примерах главы доступны только для чтения.

мы не можем заменить дочерний элемент чем то ещё путём присваивания `childNodes[i] = ...`.

Изменение DOM требует других методов. Мы рассмотрим их в следующей главе.
```

```warn header="DOM collections are live"
Almost all DOM collections with minor exceptions are *live*. In other words, they reflect the current state of DOM.

If we keep a reference to `elem.childNodes`, and add/remove nodes into DOM, then they appear in the collection automatically.
```

````warn header="Don't use `for..in` to loop over collections"
Collections are iterable using `for..of`. Sometimes people try to use `for..in` for that.

Please, don't. The `for..in` loop iterates over all enumerable properties. And collections have some "extra" rarely used properties that we usually do not want to get:

```html run
<body>
<script>
  // shows 0, 1, length, item, values and more.
  for (let prop in document.body.childNodes) alert(prop);
</script>
</body>
````

## Siblings and the parent

*Siblings* are nodes that are children of the same parent. For instance, `<head>` and `<body>` are siblings:

- `<body>` is said to be the "next" or "right" sibling of `<head>`,
- `<head>` is said to be the "previous" or "left" sibling of `<body>`.

The parent is available as `parentNode`.

The next node in the same parent (next sibling) is `nextSibling`, and the previous one is `previousSibling`.

For instance:

```html run
<html><head></head><body><script>
  // HTML is "dense" to evade extra "blank" text nodes.

  // parent of <body> is <html>
  alert( document.body.parentNode === document.documentElement ); // true

  // after <head> goes <body>
  alert( document.head.nextSibling ); // HTMLBodyElement

  // before <body> goes <head>
  alert( document.body.previousSibling ); // HTMLHeadElement
</script></body></html>
```

## Element-only navigation

Navigation properties listed above refer to *all* nodes. For instance, in `childNodes` we can see both text nodes, element nodes, and even comment nodes if there exist.

But for many tasks we don't want text or comment nodes. We want to manipulate element nodes that represent tags and form the structure of the page.

So let's see more navigation links that only take *element nodes* into account:

![](dom-links-elements.png)

The links are similar to those given above, just with `Element` word inside:

- `children` -- only those children that are element nodes.
- `firstElementChild`, `lastElementChild` -- first and last element children.
- `previousElementSibling`, `nextElementSibling` -- neighbour elements.
- `parentElement` -- parent element.

````smart header="Why `parentElement`? Can the parent be *not* an element?"
The `parentElement` property returns the "element" parent, while `parentNode` returns "any node" parent. These properties are usually the same: they both get the parent.

With the one exception of `document.documentElement`:

```js run
alert( document.documentElement.parentNode ); // document
alert( document.documentElement.parentElement ); // null
```

In other words, the `documentElement` (`<html>`) is the root node. Formally, it has `document` as its parent. But `document` is not an element node, so `parentNode` returns it and `parentElement` does not.

This loop travels up from an arbitrary element `elem` to `<html>`, but not to the `document`:
```js
while(elem = elem.parentElement) {
  alert( elem ); // parent chain till <html>
}
```
````

Let's modify one of the examples above: replace `childNodes` with `children`. Now it shows only elements:

```html run
<html>
<body>
  <div>Begin</div>

  <ul>
    <li>Information</li>
  </ul>

  <div>End</div>

  <script>
*!*
    for (let elem of document.body.children) {
      alert(elem); // DIV, UL, DIV, SCRIPT
    }
*/!*
  </script>
  ...
</body>
</html>
```

## More links: tables [#dom-navigation-tables]

Till now we described the basic navigation properties.

Certain types of DOM elements may provide additional properties, specific to their type, for convenience.

Tables are a great example and important particular case of that.

**The `<table>`** element supports (in addition to the given above) these properties:
- `table.rows` -- the collection of `<tr>` elements of the table.
- `table.caption/tHead/tFoot` -- references to elements `<caption>`, `<thead>`, `<tfoot>`.
- `table.tBodies` -- the collection of `<tbody>` elements (can be many according to the standard).

**`<thead>`, `<tfoot>`, `<tbody>`** elements provide the `rows` property:
- `tbody.rows` -- the collection of `<tr>` inside.

**`<tr>`:**
- `tr.cells` -- the collection of `<td>` and `<th>` cells inside the given `<tr>`.
- `tr.sectionRowIndex` -- the position (index) of the given `<tr>` inside the enclosing `<thead>/<tbody>/<tfoot>`.
- `tr.rowIndex` -- the number of the `<tr>` in the table as a whole (including all table rows).

**`<td>` and `<th>`:**
- `td.cellIndex` -- the number of the cell inside the enclosing `<tr>`.

An example of usage:

```html run height=100
<table id="table">
  <tr>
    <td>one</td><td>two</td>
  </tr>
  <tr>
    <td>three</td><td>four</td>
  </tr>
</table>

<script>
  // get the content of the first row, second cell
  alert( table.*!*rows[0].cells[1]*/!*.innerHTML ) // "two"
</script>
```

The specification: [tabular data](https://html.spec.whatwg.org/multipage/tables.html).

There are also additional navigation properties for HTML forms. We'll look at them later when we start working with forms.

# Summary

Given a DOM node, we can go to its immediate neighbours using navigation properties.

There are two main sets of them:

- For all nodes: `parentNode`, `childNodes`, `firstChild`, `lastChild`, `previousSibling`, `nextSibling`.
- For element nodes only: `parentElement`, `children`, `firstElementChild`, `lastElementChild`, `previousElementSibling`, `nextElementSibling`.

Some types of DOM elements, e.g. tables, provide additional properties and collections to access their content.
