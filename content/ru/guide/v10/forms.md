---
name: Формы
description: 'Как создавать потрясающие формы в Preact, которые работают где угодно.'
---

# Формы

Формы в Preact работают почти так же, как и в HTML. Вы визуализируете элемент управления и прикрепляете к нему прослушиватель событий.

Основное отличие заключается в том, что в большинстве случаев `значением` (`value`) управляет не узел DOM, а Preact.

---

<div><toc></toc></div>

---

## Контролируемые и неконтролируемые компоненты

Говоря об элементах управления формы, часто можно встретить слова «контролируемый компонент» и «неконтролируемый компонент». Описание относится к тому, как обрабатывается поток данных. DOM имеет двунаправленный поток данных, поскольку каждый элемент управления формой будет сам управлять вводом данных пользователем. Простой текстовый ввод всегда будет обновлять свое значение, когда пользователь введет в него текст.

Фреймворк типа Preact, напротив, имеет однонаправленный поток данных. Компонент управляет там не самим значением, а чем-то другим, расположенным выше в дереве компонентов.

```jsx
// Неконтролируемый, так как Preact не устанавливает значение
<input onInput={myEventHandler} />;

// Контролируемый, так как Preact теперь управляет значением входа
<input value={someValue} onInput={myEventHandler} />;
```

Как правило, следует стараться всегда использовать _контролируемые_ компоненты. Однако при создании автономных компонентов или обёртке сторонних библиотек пользовательского интерфейса может оказаться полезным просто использовать свой компонент в качестве точки монтирования для функциональности, не связанной с Preact. В этих случаях «неконтролируемые» компоненты прекрасно справляются с поставленной задачей.

> Одна загвоздка заключается в том, что при установке значения `undefined` или `null` оно, по сути, станет неуправляемым.

## Создание простой формы

Создадим простую форму для отправки элементов todo. Для этого мы создаем элемент `<form>` и привязываем обработчик событий, который будет вызываться каждый раз, когда форма будет отправлена. Аналогичным образом мы поступаем и с полем ввода текста, но обратите внимание, что мы сами храним значение в нашем классе. Вы угадали, здесь мы используем _управляемый_ вход. В данном примере это очень полезно, поскольку нам необходимо отобразить значение input в другом элементе.

```jsx
// --repl
import { render, Component } from 'preact';
// --repl-before
class TodoForm extends Component {
  state = { value: '' };

  onSubmit = (e) => {
    alert('Задача отправлена');
    e.preventDefault();
  };

  onInput = (e) => {
    this.setState({ e.target.value });
  };

  render(_, { value }) {
    return (
      <form onSubmit={this.onSubmit}>
        <input type='text' value={value} onInput={this.onInput} />
        <p>Вы ввели это значение: {value}</p>
        <button type='submit'>Отправить</button>
      </form>
    );
  }
}
// --repl-after
render(<TodoForm />, document.getElementById('app'));
```

## Элемент выбора

Элемент `<select>` немного сложнее, но работает так же, как и все остальные элементы управления формы:

```jsx
// --repl
import { render, Component } from 'preact';

// --repl-before
class MySelect extends Component {
  state = { value: '' };

  onChange = (e) => {
    this.setState({ value: e.target.value });
  };

  onSubmit = (e) => {
    alert('Отправлено ' + this.state.value);
    e.preventDefault();
  };

  render(_, { value }) {
    return (
      <form onSubmit={this.onSubmit}>
        <select value={value} onChange={this.onChange}>
          <option value='A'>A</option>
          <option value='B'>B</option>
          <option value='C'>C</option>
        </select>
        <button type='submit'>Отправить</button>
      </form>
    );
  }
}
// --repl-after
render(<MySelect />, document.getElementById('app'));
```

## Флажки и радиокнопки

Флажки и радиокнопки (`<input type="checkbox|radio">`) поначалу могут вызывать путаницу при построении управляемых форм. Это связано с тем, что в неконтролируемой среде мы обычно позволяем браузеру «переключать» или «проверять» флажок или радиокнопку, прослушивая событие изменения и реагируя на новое значение. Однако эта техника плохо переходит в картину мира, где пользовательский интерфейс всегда обновляется автоматически в ответ на изменения состояния и параметров.

> **Объяснение:** Допустим, мы прослушиваем событие `change` флажка, которое срабатывает, когда флажок устанавливается или снимается пользователем. В нашем обработчике событий изменения мы устанавливаем значение в `state`, равное новому значению, полученному от флажка. Это приведет к повторному рендерингу нашего компонента, который переназначит значение флажка значению из состояния. В этом нет необходимости, потому что мы просто запросили у DOM значение, а затем приказали ему выполнить повторную визуализацию с любым значением, которое мы хотим.

Таким образом, вместо того, чтобы прослушивать событие `input`, мы должны прослушивать событие `click`, которое запускается каждый раз, когда пользователь нажимает на флажок _или связанный с ним элемент `<label>`_. Флажки просто переключаются между логическими значениями `true` и `false`, поэтому, кликая на флажке или метке, мы просто инвертируем любое значение, которое у нас есть в состоянии, запуская повторный рендеринг и устанавливая отображаемое значение флажка на то, которое мы хотим.

### Пример флажка

```jsx
// --repl
import { render, Component } from 'preact';
// --repl-before
class MyForm extends Component {
  toggle = (e) => {
    let checked = !this.state.checked;
    this.setState({ checked });
  };

  render(_, { checked }) {
    return (
      <label>
        <input type='checkbox' checked={checked} onClick={this.toggle} />
        установите флажок
      </label>
    );
  }
}
// --repl-after
render(<MyForm />, document.getElementById('app'));
```