---
name: Модульное тестирование с помощью Enzyme
permalink: '/guide/unit-testing-with-enzyme'
description: 'Тестирование приложений Preact стало проще благодаря Enzyme'
---

# Модульное тестирование с помощью Enzyme

[Enzyme](https://airbnb.io/enzyme/) от Airbnb — это библиотека для написания тестов для компонентов React. Она поддерживает различные версии React и React-подобных библиотек с использованием «адаптеров». Существует адаптер для Preact, поддерживаемый командой Preact.

Enzyme поддерживает тесты, которые выполняются в обычном или автономном браузере с использованием такого инструмента, как [Karma](http://karma-runner.github.io/latest/index.html), или тесты, которые выполняются в Node с использованием [jsdom](https://github.com/jsdom/jsdom) как поддельная реализация API браузера.

Подробное введение в использование Enzyme и справочник по API см. в [документации по Enzyme](https://airbnb.io/enzyme/). В оставшейся части этого руководства объясняется, как настроить Enzyme с Preact, а также чем Enzyme с Preact отличается от Enzyme с React.

---

<div><toc></toc></div>

---

## Установка

Установите Enzyme и адаптер Preact, используя:

```sh
npm install --save-dev enzyme enzyme-adapter-preact-pure
```

## Конфигурация

В коде настройки теста вам необходимо настроить Enzyme для использования адаптера Preact:

```js
import { configure } from 'enzyme';
import Adapter from 'enzyme-adapter-preact-pure';

configure({ adapter: new Adapter() });
```

Рекомендации по использованию Enzyme с различными средствами запуска тестов см. в разделе [Руководства](https://airbnb.io/enzyme/docs/guides.html) документации Enzyme.

## Пример

Предположим, у нас есть простой компонент `Counter`, который отображает начальное значение с кнопкой для его обновления:

```jsx
import { h } from 'preact';
import { useState } from 'preact/hooks';

export default function Counter({ initialCount }) {
  const [count, setCount] = useState(initialCount);
  const increment = () => setCount(count + 1);

  return (
    <div>
      Текущее значение: {count}
      <button onClick={increment}>Увеличить</button>
    </div>
  );
}
```

Используя средство запуска тестов, такое как Mocha или Jest, вы можете написать тест, чтобы убедиться, что он работает должным образом:

```jsx
import { expect } from 'chai';
import { h } from 'preact';
import { mount } from 'enzyme';

import Counter from '../src/Counter';

describe('Counter', () => {
  it('должен отображать начальный счётчик', () => {
    const wrapper = mount(<Counter initialCount={5} />);
    expect(wrapper.text()).to.include('Текущее значение: 5');
  });

  it('значение должно увеличиваться после нажатия кнопки «Увеличить»', () => {
    const wrapper = mount(<Counter initialCount={5} />);

    wrapper.find('button').simulate('click');

    expect(wrapper.text()).to.include('Текущее значение: 6');
  });
});
```

Работоспособную версию этого проекта и другие примеры можно найти в каталоге [examples/](https://github.com/preactjs/enzyme-adapter-preact-pure/blob/master/README.md#example-projects) в репозитории адаптера Preact.

## Как работает Enzyme

Enzyme использует библиотеку адаптеров, с которой он был настроен, для рендеринга компонента и его дочерних элементов. Затем адаптер преобразует выходные данные в стандартизированное внутреннее представление («Стандартное дерево React»). Затем Enzyme оборачивает это объектом, имеющим методы для запроса выходных данных и запуска обновлений. API объекта-обертки использует CSS-подобные [селекторы](https://airbnb.io/enzyme/docs/api/selector.html) для поиска частей выходных данных.

## Полный, поверхностный и строковый рендеринг

Enzyme имеет три «режима» рендеринга:

```jsx
import { mount, shallow, render } from 'enzyme';

// Отображаем полное дерево компонентов:
const wrapper = mount(<MyComponent prop='value' />);

// Отображаем только прямой вывод `MyComponent` (т. е. «имитация» дочерних компонентов
// для рендеринга только в качестве заполнителей):
const wrapper = shallow(<MyComponent prop='value' />);

// Отображаем полное дерево компонентов в строку HTML и анализируем результат:
const wrapper = render(<MyComponent prop='value' />);
```

- Функция `mount` отображает компонент и всех его потомков так же, как они отображались бы в браузере.

- Функция `shallow` отображает только те узлы DOM, которые непосредственно выводятся компонентом. Любые дочерние компоненты заменяются заполнителями, которые выводят только их дочерние элементы.

  Преимущество этого режима в том, что вы можете писать тесты для компонентов, не вдаваясь в подробности дочерних компонентов и не создавая всех их зависимостей.

  Режим `shallow` («поверхностного») рендеринга в адаптере Preact работает иначе, чем в React. Подробности смотрите в разделе «Различия» ниже.

- Функция `render` (не путать с функцией `render` Preact!) отображает компонент в HTML-строку. Это полезно для тестирования результатов рендеринга на сервере или рендеринга компонента без запуска каких-либо его эффектов.

## Запуск обновлений состояния и эффектов с помощью `act`

В предыдущем примере `.simulate('click')` использовался для нажатия кнопки.

Enzyme знает, что вызовы `simulate` могут изменить состояние компонента или вызвать эффекты, поэтому он будет применять любые обновления состояния или эффекты непосредственно перед возвратом `simulate`. Enzyme делает то же самое, когда компонент изначально визуализируется с использованием `mount` или `shallow` и когда компонент обновляется с помощью `setProps`.

Однако если событие происходит вне вызова метода Enzyme, например, при прямом вызове обработчика событий (например, свойства `onClick` кнопки), то Enzyme не будет знать об изменении. В этом случае вашему тесту потребуется инициировать выполнение обновлений состояния и эффектов, а затем попросить Enzyme обновить представление выходных данных.

- Чтобы синхронно выполнять обновления состояния и эффекты, используйте функцию `act` из `preact/test-utils` для обертывания кода, запускающего обновления
- Для обновления представления Enzyme о выводимых данных используйте метод обертки `.update()`.

Например, здесь представлена другая версия теста для увеличения счётчика, модифицированная для прямого вызова свойства `onClick` кнопки, а не через метод `simulate`:

```js
import { act } from 'preact/test-utils';
```

```jsx
it('значение должно увеличиваться после нажатия кнопки «Увеличить»', () => {
  const wrapper = mount(<Counter initialCount={5} />);
  const onClick = wrapper.find('button').props().onClick;

  act(() => {
    // Вызываем обработчик нажатия кнопки, но на этот раз напрямую, а не через Enzyme API
    onClick();
  });
  // Обновляем представление результатов Enzyme
  wrapper.update();

  expect(wrapper.text()).to.include('Текущее значение: 6');
});
```

## Отличия от Enzyme с React

Общая цель состоит в том, чтобы тесты, написанные с использованием Enzyme + React, можно было легко заставить работать с Enzyme + Preact или наоборот. Это позволяет избежать необходимости переписывать все ваши тесты, если вам нужно переключить компонент, изначально написанный для Preact, на работу с React или наоборот.

Однако существуют некоторые различия в поведении этого адаптера и адаптеров Enzyme React, о которых следует знать:

- «Поверхностный» режим рендеринга (`shallow`) работает по-другому. Он совместим с React, поскольку отображает компонент только «на один уровень глубины», но, в отличие от React, создает настоящие узлы DOM. Он также запускает все обычные перехватчики и эффекты жизненного цикла.
- Метод `simulate` отправляет реальные события DOM, тогда как в адаптерах React `simulate` просто вызывает свойство `on<EventName>`.
- В Preact обновления состояния (например, после вызова `setState`) группируются и применяются асинхронно. В состоянии React обновления могут применяться немедленно или пакетно в зависимости от контекста. Чтобы упростить написание тестов, адаптер Preact сбрасывает обновления состояния и эффекты после начального рендеринга и обновлений, запускаемых с помощью вызовов `setProps` или `simulate` на адаптере. Когда обновления состояния или эффекты запускаются другими способами, вашему тестовому коду может потребоваться вручную запустить очистку эффектов и обновлений состояния с помощью `act` из пакета `preact/test-utils`.

Дополнительные сведения см. в [README адаптера Preact](https://github.com/preactjs/enzyme-adapter-preact-pure#differences-compared-to-enzyme--react).