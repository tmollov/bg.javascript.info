# Копиране на обекти, референции

Една от основните разлики между обектите и примитивите типове е, че те се съхраняват и копират "чрез референция".

Примитивите типове: низове, числа, булеви -- се присвояват/копират "като цялостна стойност".

Например:

```js
let message = "Hello!";
let phrase = message;
```

В резултат на това имаме две независими променливи, всяка от които съхранява низ `"Hello!"`.

![](variable-copy-value.svg)

Обектите не са такива.

**Променливата съхранява не самия обект, а "адреса в паметта", с други думи "референцията" към него.**

Ето снимката на обекта:

```js
let user = {
  name: "John"
};
```

![](variable-contains-reference.svg)

Тук обектът се съхранява някъде в паметта. И променливата `user` има "референция" към него.

**Когато обектната променлива е копирана -- референцията е копирана, обектът не се дублира.**

Например:

```js no-beautify
let user = { name: "John" };

let admin = user; // копира референцията
```

Сега имаме две променливи, всеки от тях с референция към един и същ обект:

![](variable-copy-reference.svg)

Можем да използваме всяка променлива за достъп до обекта и да променяме съдържанието му:

```js run
let user = { name: 'John' };

let admin = user;

*!*
admin.name = 'Pete'; // променен от "admin" референцията
*/!*

alert(*!*user.name*/!*); // 'Pete', промените се виждат от "user" референцията
```

Примерът по-горе демонстрира, че има само един обект. Сякаш имахме шкаф с два ключа и използвахме един от тях (`admin`), за да влезем в него. След това, ако по-късно използваме друг ключ (`user`) можем да видим промените.

## Сравнение по референция

Операторите за равенството `==` и строгото равенство `===` работят по същия начин и за обектите.

**Два обекта са равни само ако са един и същ обект.**

Тук две променливи се отнасят към един и същ обект, като по този начин те са равни:

```js run
let a = {};
let b = a; // копира референцията

alert( a == b ); // true, и двете променливи се отнасят към един и същ обект
alert( a === b ); // true
```

И тук два независими обекта не са равни, въпреки че и двата са празни:

```js run
let a = {};
let b = {}; // два независими обекта

alert( a == b ); // falsе
```

За сравнения като `obj1 > obj2` или за сравнение с примитив `obj == 5`, обектите се преобразуват в примитиви. Ще изучим как преобразуванията на обекти работят много скоро, но за да кажем истината, подобни сравнения се случват много рядко, обикновено в резултат на грешка в кода.

## Клониране и сливане, Object.assign

Така, копирането на променливата на обекта създава още една референция към същия обект.

Но какво ще стане, ако трябва да дублираме обект? Да създадете независимо копие, клонинг?

Това също е изпълнимо, но малко по-трудно, тъй като няма вграден метод за това в JavaScript. Всъщност това рядко е необходимо. Копирането чрез референция е добро през повечето време.

Но ако наистина искаме това, тогава трябва да създадем нов обект и да репликираме структурата на съществуващия, като повтаряме неговите свойства и ги копираме на примитивно ниво.

Като този:

```js run
let user = {
  name: "John",
  age: 30
};

*!*
let clone = {}; // новият празен обект

//нека копираме всички свойства на потребителя в него
for (let key in user) {
  clone[key] = user[key];
}
*/!*

// сега клонингът е напълно независим обект със същото съдържание
clone.name = "Pete"; // променяме данните в него

alert( user.name ); // все още "John" e в оригиналния обект
```

Също така можем да използваме метода [Object.assign](mdn:js/Object/assign) за този цел.

Синтаксисът е:

```js
Object.assign(dest, [src1, src2, src3...])
```

- Първият аргумент `dest` е целевия обект.
- Допълнителните аргументи `src1, ..., srcN` (могат да бъдат колкото са необходими) са обекти-източници.
- Той копира свойствата на всички обекти-източници `src1, ..., srcN` в целта `dest`.С други думи, свойствата на всички аргументи, започвайки от втория, се копират в първия обект.
- Функцията връща `dest`.

Например, можем да го използваме за сливане на няколко обекта в един:
```js
let user = { name: "John" };

let permissions1 = { canView: true };
let permissions2 = { canEdit: true };

*!*
// копира всички свойства от permissions1 и permissions2 в user
Object.assign(user, permissions1, permissions2);
*/!*

// сега user = { name: "John", canView: true, canEdit: true }
```

Ако името на копираното свойство вече съществува, то се презаписва:

```js run
let user = { name: "John" };

Object.assign(user, { name: "Pete" });

alert(user.name); // сега user = { name: "Pete" }
```

Също можем да използваме `Object.assign` за да заменим `for..in` цикъла за просто клониране:

```js
let user = {
  name: "John",
  age: 30
};

*!*
let clone = Object.assign({}, user);
*/!*
```

Той копира всички свойства на `user`в празния обект и го връща.

## Вложено клониране

Досега приемахме, че всички свойства на `user` са примитивни. Но свойствата могат да бъдат референции към други обекти. Какво да правим с тях?

Като тези тук:
```js run
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};

alert( user.sizes.height ); // 182
```

Сега не е достатъчно да копираме `clone.sizes = user.sizes`, защото `user.sizes` е обект, и ще се копира чрез референция. Така `clone` и `user` ще споделят същите размери:

Като тук:

```js run
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};

let clone = Object.assign({}, user);

alert( user.sizes === clone.sizes ); // true, същия обект

// "user" и "clone" споделят размерите
user.sizes.width++;       // променяте свойсвото от едно място
alert(clone.sizes.width); // 51, виждате резултата от друго
```

За да поправим това, трябва да използваме клониращия цикъл, който изследва всяка стойност на `user[key]` и, ако е обект, след това репликирайте и неговата структура. Това се нарича "deep cloning" (т.н. "дълбоко клониране").

Има стандартен алгоритъм за дълбоко клониране, който обработва случая по-горе и по-сложни случаи, наречени the[Структуриран алгоритъм на клониране](https://html.spec.whatwg.org/multipage/structured-data.html#safe-passing-of-structured-data).

Можем да използваме рекурсия за нейното имплементиране. Или, за да не изобретим колелото отново, вземете съществуваща имплементация, например [_.cloneDeep(obj)](https://lodash.com/docs#cloneDeep) от JavaScript библеотеката [lodash](https://lodash.com).

## Обобщение

Обектите се присвояват и копират чрез референция. С други думи, променливата съхранява "референцията" към стойността на обекта (адреса в паметта), а не самата "стойност на обекта". Така че, копирането на такава променлива или предаването й като аргумент на функция копира тази референция, а не самия обекта.

Всички операции чрез копирани референции (като добавяне/изтриване на свойства) се изпълняват върху един и същ обект.

За да направим "реално копие" (клонинг) можем да използваме `Object.assign` за т.н. "плитко копие" (вложените обекти се копират чрез референция) или функции за "дълбоко клониране", като [_.cloneDeep(obj)](https://lodash.com/docs#cloneDeep).
