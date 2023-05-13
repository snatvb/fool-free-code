# Принцип контекстов

_Объединяй и властвуй_

Очень часто можно наблюдать, как разработчики пишут код в таком духе:

```ts
export const findUser = (id: number) => {};
export const checkUserExists = (id: number) => {};
export const userValidate = (user: User) => {};
export const getUserName = (id: number) => {};
```

Проблема в том, что здесь явно прослеживается связь функций - это сущность **User**. Это означает, что можно эти функции объединить под один контекст.

```ts
export const user = {
    find(id: number) {},
    checkExists(id: number) {},
    validate(user: User) {},
    getName(id: number) {},
};
```

`as user from ...`, вот пример файла `user/user.ts`:

```ts
export const find = (id: number) => {};
export const checkExists = (id: number) => {};
export const validate = (user: User) => {};
export const getName = (id: number) => {};
```

Файл `user/index.ts`:

```ts
export * as user from './user';
```

Теперь это можно использовать следующим образом:

```ts
const name = user.getName(id);

// ...

if (user.validate(admin)) {
}
```

## Кошелечек в сумочке, сумочка в мешочке

Контексты могут быть вложенными! Вложенные контексты очень удобны. Представим такую ситуацию, что у нас несколько разных сущностей - `User`, `Post`, `Comment`

Но над ними могут быть наборы разных функций, которые под собой будут иметь единый контекст. Например функции-хелперы, селекторы, события, хранилища. Здесь может быть два вида вложения.

```
helpers.user.someFunction / helpers.post.someFunction ...
store.users / store.posts / store.comments
selectors.users / selectors.posts / selectors.comments
```

Второй вариант:

```
users.selectors. / users.herlpers. / users.store /
posts.selectors. / posts.herlpers. / posts.store /
comments.selectors. / comments.herlpers. / comments.store /
```

Два варианта никак не противоречат друг другу. Более того, иногда можно встретить обе варианта в одном проекте и это не будет считаться ошибкой. Например в какой то момент store может быть неудобно хранить в сущности users / posts / comments и потребуется инвертирование, более того, могут накладываться ограничения используемых технологий.

## Эй вы, трое, а ну сюда оба подошли, да, я тебе говорю!

Часто возникают проблемы с тем, а как правильно именовать сущности - во множественном или единственном числе. Например как user в контексте. Если речь идет про какое-то хранилище, ответ довольно прост. Задается вопрос - `здесь будет храниться один элемент или несколько?`, если несколько, то используется множественное, иначе единственное. Здесь существует несколько решений.

### Проще простого

Первое - именование происходит от одиночной сущности и там где требуется множественное, используется префикс или постфикс:

```ts
// user.ts

export const find = (id: number) => {};
export const validate = (user: User) => {};
export const getName = (id: number) => {};

// many
export const findMany = (ids: number[]) => {};
export const getNames = (ids: number[]) => {};
export const everyValid = (users: User[]) => {};
```

### Шиворот на выворот

Второй - обратное первому, всегда используется множественное, но тогда единственное число будет префикс или постфикс у функций, которые оперируют единичными сущностями:

```ts
// users.ts

export const find = (ids: number[]) => {};
export const validate = (user: User[]) => {};
export const getName = (ids: number[]) => {};

// single
export const findOne = (id: number) => {};
export const getNameOne = (id: number) => {};
export const validOne = (users: User) => {};
```

Как можно заметить, данный метод менее лаконичный. Более того, он более сложный при чтении. Программисты уже привыкли оперировать сущностями в единственном числе. К тому же, как можно заметить, часто можно применять синонимы для функций, которые дают понять, что здесь выработаете с множеством. В обратную сторону это практически не работает.

### И волки сыты и овцы целы

Третий вариант - разбить контекст на два. На самом деле у вас может сложиться впечатление, что это лишнее, но это не так. Лично мне он нравится гораздо больше двух предыдущих. Когда я работаю с этим вариантом, я начинаю разделять сущности и коллекции сущностей. Например пользователь это одна сущность, а пользователи - уже другая. Пример на типах:

```ts
type User = {
    id: number;
    firstName: string;
    lastName: string;
};

type Users = Record<number, User>;
type UserList = User[];
```

Соответственно по данному примеру у меня будет примерно такая структура:

```ts
selectors.user.find(id);
selectors.users.find(ids);
selectors.userList.find(ids);

helpers.userList.convert.toUsers(userList); // -> Record<number, User>
```

Мне кажется это самым правильным и чистым решением.
