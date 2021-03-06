# Подготовьте данные для слайдов
## Задача

Мы будем брать данные из Code Hub. Команда разработчиков уже подготовила инновационный бэкенд, работающий с данными Code Hub. Данные — это записи о пользователях и их действиях — коммитах, комментариях, заведённых проблемах (issues), а также спринтах.

Наша задача — преобразовывать ответ от бэкенда в более простой формат данных, пригодный для рендеринга слайдов. Это будут максимально простые структуры, которые легко шаблонизировать.

В качестве решения нужно предоставить файл `index.js`, содержащий функцию `prepareData`. Функция должна принимать на вход массив данных, полученный с бэкенда и объект, содержащий id спринта (числовое поле `sprintId`). Она должна возвращать массив объектов, каждый из которых описывает отдельный слайд.

```ts
function prepareData(entities: Entity[], { sprintId: number }): Slide[] {
    // ...
}

module.exports = { prepareData }
```

Ваш код должен не зависеть от платформы (браузер/node.js) и не требовать подключения дополнительных зависимостей. Если файл является результатом сборки, то предоставьте также исходный код, из которого он был собран.

## Формат входных данных

Бекэнд оперирует следующими сущностями из Code Hub:

1. проект (репозиторий);
2. пользователь;
3. коммит;
4. отдельный файл в коммите;
5. проблема (issue);
6. комментарий к проблеме;
7. спринт — период работы команды разработчиков, который длится одну неделю. Задачи команды планируются по спринтам. Спринты идут друг за другом и нумеруются по порядку.

Бэкенд присылает данные в виде массива объектов. Каждый его элемент — это одна из сущностей Code Hub. Каждая сущность имеет поле `type`, по значению которого можно понять, какого она типа. В массиве могут находиться сущности разных типов, например:

```ts
[
    { type: 'Project', ... }
    { type: 'Sprint', ... }
    { type: 'Issue', ... }
]
```

Сущности Code Hub могут ссылаться друг на друга. Например, в каждом коммите (commit) есть информация о его авторе (user), а каждый комментарий (comment) ссылается на проблему (issue), к которой он относится. В качестве ссылки может быть указан как id связанной сущности, так и объект, содержащий все её поля.

Пример:

```ts
[
    {
        type: 'User',
        friends: [
            '1', // пользователь ссылается на другого пользователеля по id
            {
                type: 'User',  // полная информация о связанном пользователе
                friends: [...]
            }
        ] 
    }
]
```

Обратите внимание, вложенные объекты тоже могут содержать ссылки на вложенные объекты и образовывать несколько уровней вложенности.

Узнайте подробнее о формате входных данных:

- [полное описание полей](./examples/types.d.ts) для каждой сущности;
- [полный пример ответа бэкенда](https://drive.google.com/file/d/1X_H37zr9vB4yLbmV6Dl-vj3xyfa59zjd/view?usp=sharing).

## Формат выходных данных

Каждый слайд визуализирует небольшой объём информации при помощи одного из пяти шаблонов. Подробности о различных типах слайдов вы можете узнать [описания типов данных](./examples/format.md).

### 1. Самый внимательный разработчик

Голосование за участника команды — выводит список участников команды постранично и позволяет проголосовать за участника в одной из номинаций.

<details>
<summary><b>Скриншот</b></summary>
<img src="./assets/img/attention.png" style="max-width: 193px"/>
</details>

Нужно найти все лайки к комментариям, полученные разработчиками за спринт, и просуммировать их количество.

<details>
<summary><b>Формат слайда</b></summary>
<pre>
{
  "alias": "vote",
  "data": {
    "title": "Самый 🔎 внимательный разработчик",
    "subtitle": "Спринт № 213",
    "emoji": "🔎",
    "users": [
      {"id": 1, "name": "Евгений Дементьев", "avatar": "1.jpg", "valueText": "22 голоса"},
      {"id": 4, "name": "Вадим Пацев", "avatar": "4.jpg", "valueText": "19 голосов"},
      {"id": 10, "name": "Яна Берникова", "avatar": "10.jpg", "valueText": "17 голосов"},
      {"id": 12, "name": "Алексей Ярошевич", "avatar": "12.jpg", "valueText": "16 голосов"},
      {"id": 11, "name": "Юрий Фролов", "avatar": "11.jpg", "valueText": "11 голосов"},
      {"id": 2, "name": "Александр Шлейко", "avatar": "2.jpg", "valueText": "10 голосов"},
      {"id": 5, "name": "Александр Николаичев", "avatar": "5.jpg", "valueText": "9 голосов"},
      {"id": 6, "name": "Андрей Мокроусов", "avatar": "6.jpg", "valueText": "8 голосов"},
      {"id": 8, "name": "Александр Иванков", "avatar": "8.jpg", "valueText": "7 голосов"},
      {"id": 7, "name": "Дмитрий Андриянов", "avatar": "7.jpg", "valueText": "6 голосов"},
      {"id": 3, "name": "Дарья Ковалева", "avatar": "3.jpg", "valueText": "5 голосов"},
      {"id": 9, "name": "Сергей Бережной", "avatar": "9.jpg", "valueText": "4 голоса"}
    ]
  }
}
</pre>
</details>

### 2. Лидеры по коммитам

Лидеры спринта — позволяет увидеть участников команды, лидирующих в разных номинациях. Также используется для отображения результатов голосования.

<details>
<summary><b>Скриншот</b></summary>
<img src="./assets/img/leader.png" style="max-width: 193px"/>
</details>

Вам нужно найти все коммиты из заданного спринта, сгруппировать их по пользователям.

<details>
<summary><b>Формат слайда</b></summary>
<pre>
{
  "alias": "leaders",
  "data": {
    "title": "Больше всего коммитов",
    "subtitle": "Спринт № 213",
    "emoji": "👑",
    "users": [
      {"id": 3, "name": "Дарья Ковалева", "avatar": "3.jpg", "valueText": "32"},
      {"id": 9, "name": "Сергей Бережной", "avatar": "9.jpg", "valueText": "27"},
      {"id": 7, "name": "Дмитрий Андриянов", "avatar": "7.jpg", "valueText": "22"},
      {"id": 6, "name": "Андрей Мокроусов", "avatar": "6.jpg", "valueText": "20"},
      {"id": 8, "name": "Александр Иванков", "avatar": "8.jpg", "valueText": "19"}
    ]
  }
}
</pre>
</details>

### 3. Коммиты

Статистика по спринтам — визуализирует статистику текущего спринта по сравнению с предыдущими спринтами, например показывает количество коммитов в текущем и предыдущих спринтах.

<details>
<summary><b>Скриншот</b></summary>
<img src="./assets/img/commits.png" style="max-width: 193px"/>
</details>

В верхней части слайда отображается диаграмма, которая показывает количество коммитов в каждом спринте. В нижней части находится список пользователей с наибольшим числом коммитов в текущем спринте.

<details>
<summary><b>Формат слайда</b></summary>
<pre>
{
    "alias": "chart",
    "data": {
      "title": "Коммиты",
      "subtitle": "Спринт № 213",
      "values": [
        {"title": "208", "value": 152},
        {"title": "209", "value": 128},
        {"title": "210", "value": 164},
        {"title": "211", "value": 118},
        {"title": "212", "value": 140},
        {"title": "213", "value": 182, "active": true},
        {"title": "214", "value": 0},
        {"title": "215", "value": 0},
        {"title": "216", "value": 0},
      ],
      "users": [
        {"id": 3, "name": "Дарья Ковалева", "avatar": "3.jpg", "valueText": "32"},
        {"id": 9, "name": "Сергей Бережной", "avatar": "9.jpg", "valueText": "27"},
        {"id": 7, "name": "Дмитрий Андриянов", "avatar": "7.jpg", "valueText": "22"}
      ]
    }
  },
</pre>
</details>

### 4. Размер коммитов

Статистика внутри спринта — визуализирует статистику внутри спринта по качественным характеристикам, например размеру коммитов в спринте, и разницу с предыдущим спринтом.

<details>
<summary><b>Скриншот</b></summary>
<img src="./assets/img/size.png" style="max-width: 193px"/>
</details>

Вам нужно найти все коммиты в текущем спринте и сгруппировать из по размеру (см. макет). После этого сделайте всё то же самое для предыдущего спринта и посчитайте разницу между ними (как по группам, так и общую - см. макет).

Размер коммита — это сумма количества добавленных и удаленных строк. Например, если добавлено 5 строк и удалено 5 строк, то размер коммита будет равен 10.

<details>
<summary><b>Формат слайда</b></summary>
<pre>
{
  "alias": "diagram",
  "data": {
    "title": "Размер коммитов",
    "subtitle": "Спринт № 213",
    "totalText": "182 коммита",
    "differenceText": "+42 с прошлого спринта",
    "categories": [
      {"title": "> 1001 строки", "valueText": "30 коммитов", "differenceText": "+8 коммитов"},
      {"title": "501 — 1000 строк", "valueText": "32 коммита", "differenceText": "+6 коммитов"},
      {"title": "101 — 500 строк", "valueText": "58 коммитов", "differenceText": "+16 коммитов"},
      {"title": "1 — 100 строк", "valueText": "62 коммита", "differenceText": "+12 коммитов"}
    ]
  }
}
</pre>
</details>

### 5. Активность

Активность участников — визуализирует активность команды в виде тепловой карты по дням и часам.

<details>
<summary><b>Скриншот</b></summary>
<img src="./assets/img/activity.png" style="max-width: 193px"/>
</details>

Нужно сгруппировать коммиты (для всех пользователей) в текущем спринте по дате и времени.

<details>
<summary><b>Формат слайда</b></summary>
<pre>
{
  "alias": "activity",
  "data": {
    "title": "Коммиты, 1 неделя",
    "subtitle": "Спринт № 213",
    "data": {
      "mon": [0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 2, 3, 2, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0],
      "tue": [0, 0, 0, 0, 1, 0, 0, 0, 0, 5, 0, 4, 0, 0, 0, 0, 1, 0, 3, 0, 0, 2, 1, 0],
      "wed": [1, 0, 0, 0, 1, 0, 5, 0, 0, 4, 0, 0, 0, 5, 0, 2, 1, 0, 0, 0, 0, 0, 0, 1],
      "thu": [0, 1, 0, 1, 0, 0, 0, 0, 6, 0, 1, 0, 0, 1, 0, 0, 5, 0, 0, 0, 1, 0, 0, 0],
      "fri": [0, 0, 0, 0, 0, 0, 0, 1, 3, 0, 0, 5, 0, 4, 0, 0, 3, 0, 0, 0, 0, 1, 0, 0],
      "sat": [0, 0, 0, 0, 2, 1, 0, 0, 1, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0],
      "sun": [0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 3, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0]
    }
  }
}
</pre>
</details>
