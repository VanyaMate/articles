# Frontend. MVE архитектура

Здравствуйте.

Из этой статьи вы узнаете об определенном архитектурном подходе, который я назвал **MVE**.

Вы, возможно, узнаете в данном подходе **Flux** или **MVI**, но я считаю, что это что-то другое. Почему - вы узнаете.

Так же весь код считайте псевдокодом.

****

## Предисловие

Перед тем как начать, важно понимать пару вещей которые я для себя решил считать верными и следую им.

1.

Архитектуры, библиотеки, фреймворки, принципы, тестирование, итд. - это инструменты и у каждого есть своя область
применения, свои плюсы и минусы, своя цена итд.

- Какие-то инструменты много весят, но дают большой функционал.
- Какие-то дают вам удобство, но забирают производительность.
- Какие-то ограничивают вас, но делают проект легко изменяемым.
- Какие-то забирают больше времени, но делают так что проект не умрет через день.
- итд.

И каждый инструмент нужен тогда когда нужен именно такой инструмент. Как бы это очевидно не звучало.

2.

Так же я всегда стараюсь всё максимально упростить с точки зрения абстрактности, чтобы уместить приложение в голове
целиком, каким бы большим оно не было.

Сделать сложно - очень просто, но сделать просто - очень сложно.

****

## Архитектура MVE

Что такое **MVE**? **Model**, **View**, **Effect**.

- **Model** - какое-то хранилище данных которое само себя обновляет
- **View** - какое-то отображение данных
- **Effect** - какое-то действие

Дальше будут примеры, мы всё разберем подробнее и конкретнее.

![img.png](img.png)

Для начала давайте расскажу как связаны эти сущности друг с другом.

1. **View** подписывается на изменения **Model**

Всю задачу **View** сводим к - просто показать актуальные данные. Мы подписываемся на изменения данных и при
необходимости ререндерим их. Это вся связь **View** и **Model**.

2. **Model** подписывается на **Effect**

Всю задачу **Model** сводим к - обновить саму себя в зависимости от вызванных действий. Мы подписываемся на вызовы
действий и в зависимости от действия, от его аргументов, от успешности и возвращаемых данных - модель сама себя меняет.
Это вся связь **Model** и **Effect**

3. **Anyone** вызывает **Effect**

Это значит, что действия могут вызываться кем угодно откуда угодно.

4. **Effect** оборачивает **Action**

А вот тут давайте по подробнее. Что такое **Effect**, что такое **Action** и в чем разница.

### Action

**Action** - это любое самодостаточное действие, например: "Войти в аккаунт", "Загрузить посты", "Отправить
комментарий".

```typescript
// "Войти в аккаунт"
export const signIn = (login: string, password: string) =>
    fetch(`${ __API__ }/v1/authorization`, { method: 'POST', body: JSON.stringify({ login, password }) })
        .then((response) => response.json());

// "Загрузить посты"
export const getPosts = (userId: string, limit: number) =>
    fetch(`${ __API__ }/v1/posts/user/${ userId }?limit=${ limit }`)
        .then((response) => response.json());

// "Отправить комметарий"
export const sendComment = (postId: string, comment: string) =>
    fetch(`${ __API__ }/v1/comment/${ postId }`, { method: 'POST', body: comment })
        .then((response) => response.json());

// Я намеренно упрощаю максимально код, чтобы это не отвлекало, потому что это не важно.
```

### Effect

**Effect** - это обертка над **Action** которая имеет туже сигнатуру, но на неё можно подписаться, а так же создать на
основе одного **Action** множество **Effect**.

****

Описательная часть закончилась, давайте к конкретике.

## Инструменты

Одним описанием не отделаться, разумеется. Вряд ли кто-то действительно уже понял всё о чем я хочу рассказать.

Я еще не писал о плюсах и минусах этой архитектуры, но в будущем итоги подведем.

Пока что давайте реализуем эту архитектуру и посмотрим всё на примерах, но для начала обозначим задачу и свои хотелки.

Как разработчик я хочу иметь:

- Максимальное удобство, простоту и скорость разработки

На сайте я хочу получить:

- Форму входа/регистрации
- Возможность создавать посты
- Возможность оставлять комментарии

Я не просто так писал в начале о том, что инструменты имеют свою цену, функционал и что они нужны тогда когда они нужны.
Так что нам понадобится?

****

Для своего удобства в качестве архитектуры я выберу **MVE**

****

Так же для удобства и скорости хотелось бы использовать какой-нибудь фреймворк или библиотеку.

Что нам нужно от этого фреймворка или библиотеки?

- Возможность подписываться на **Model**
- Удобный синтаксис
- Хуки для управления локальным состоянием

Возьмем несколько вариантов.

- React (40 kb gzip, JSX)
- Solid (6 kb gzip, JSX)
- Svelte (2 kb gzip)

Все они нам подходят по нашим хотелкам.

Тогда давайте возьмем Solid. Я готов заплатить 4 kb за дополнительный свой комфорт.

Отлично, с **View** разобрались.

****

Давайте теперь подумаем кто может подойти на роль **Model**.

Что нам нужно от этого инструмента?

- Возможность подписываться на **Action** создавая **Effect**
- Самообновление по подписке на вызов **Effect**

Возьмем пару вариантов:

- Redux Toolkit + Redux Thunk
- Effector

Redux Toolkit слишком много весит для такой задачи, а так же будет дополнительный код в обертке.
Effector гораздо меньше, простоё обертывание. В общем подходит. Но. Effector содержит в себе гораздо больше чем нам
нужно, а следовательно весит гораздо больше чем тот вариант который нам нужен, так что давайте найдем другой вариант.

Вариант такой: Напишем свою реализацию с двумя функциями.

1. store - создает стор у которого будет способ подписки на **Effect**
2. effect - создает **Effect** обертку над **Action**

Итого:

- MVE
- Solid
- [Своя реализация стора](https://www.npmjs.com/package/@vanyamate/sec)

## Реализация

Реализовывать верстку, полностью компоненты - мы не будем, мы будем просто обозначать что и где находится.
Структура проекта у нас будет такой:

```
- src
    - action
        - [имя].action.ts
    - model
        - [имя].model.ts
    - component
        - [тип]
            - [принадлежность к]
                - [имя].tsx
```

Давайте для начала создадим **Action**-ы.

```typescript
export type AuthResponse = {
    user: User;
    tokens: [ string, string ];
}
```

```typescript
// /src/action/signUp.action.ts
export const signUpAction = (login: string, password: string): Promise<AuthResponse> =>
    fetch(`${ __API__ }/v1/registration`, { method: 'POST', body: JSON.stringify({ login, password }) })
        .then((response) => response.json());
```

```typescript
// /src/action/signIn.action.ts
export const signInAction = (login: string, password: string): Promise<AuthResponse> =>
    fetch(`${ __API__ }/v1/authorization`, { method: 'POST', body: JSON.stringify({ login, password }) })
        .then((response) => response.json());
```

```typescript
// /src/action/getPosts.action.ts
export const getPostsAction = (userId: string, limit: number): Promise<Post[]> =>
    fetch(`${ __API__ }/v1/posts/user/${ userId }?limit=${ limit }`)
        .then((response) => response.json());
```

```typescript
// /src/action/createPost.action.ts
export const createPostAction = (post: string): Promise<Post> =>
    fetch(`${ __API__ }/v1/post`, { method: 'POST', body: post })
        .then((response) => response.json());
```

```typescript
// /src/action/sendComment.action.ts
export const sendCommentAction = (postId: string, comment: string): Promise<Comment> =>
    fetch(`${ __API__ }/v1/comment/${ postId }`, { method: 'POST', body: comment })
        .then((response) => response.json());
```

Как вы видите это просто какие-то самодостаточные функции которые просто что-то делают

****

Отлично, теперь создадим нашу модель.

Но перед этим небольшой гайдик про "свой стор" для лучшего понимания.

- effect - создает обертку
- store - создает store
- store имеет метод `on` где:
    - первый атрибут - effect
    - второй атрибут - состояние (onBefore, onSuccess, onError, onFinally)
    - третий атрибут - callback который должен вернуть новое состояние
        - callback принимает первым атрибутом прошлое состояние
        - callback принимает вторым атрибутом объект с аргументами (args) с которым вызван effect, результат (result),
          ошибка (error)

```typescript
// /src/model/auth.model.ts
import { effect, store } from '@vanyamate/sec';


export const signUpEffect = effect(signUpAction);
export const signInEffect = effect(signInAction);


export const $authIsPending = store<boolean>(false)
    .on(signUpEffect, 'onBefore', () => true)
    .on(signInEffect, 'onBefore', () => true)
    .on(signUpEffect, 'onFinally', () => false)
    .on(signInEffect, 'onFinally', () => false);

export const $authError = store<Error | null>(null)
    .on(signUpEffect, 'onError', (_, { error }) => error)
    .on(signInEffect, 'onError', (_, { error }) => error);

export const $authData = store<User | null>(null)
    .on(signUpEffect, 'onSuccess', (_, { result }) => result.user)
    .on(signInEffect, 'onSuccess', (_, { error }) => result.user);
```

```typescript
// /src/model/posts.model.ts
import { effect, store } from '@vanyamate/sec';


export const getPostsEffect   = effect(getPostsAction);
export const createPostEffect = effect(createPostAction);


export const $postsIsPending = store<boolean>(false)
    .on(getPostsEffect, 'onBefore', () => true)
    .on(getPostsEffect, 'onFinally', () => false);

export const $postsError = store<Error | null>(null)
    .on(getPostsEffect, 'onError', (_, { error }) => error);

export const $postsList = store<Post[]>([])
    .on(getPostsEffect, 'onSuccess', (_, { result }) => result);
```

// TODO (Кажется надо переписать пример. Сделать простым его не получится, а нужен простой)