# 🌈 JavaScript Front-End Cheat Sheet

## 📦 Основы синтаксиса

```js
// Переменные
let name = "Alice";
const PI = 3.14;
var age = 30;

// Типы данных
let number = 42;
let str = "Hello!";
let bool = true;
let arr = [1, 2, 3];
let obj = { name: "Bob", city: "Paris" };
```

---

## 🧩 Функции

```js
// Обычная функция
function sum(a, b) {
  return a + b;
}

// Стрелочная функция
const greet = name => `Hi, ${name}!`;

// Коллбэк
arr.forEach(item => console.log(item));
```

---

## 🔄 Управляющие конструкции

```js
if (age > 18) {
  console.log("Adult");
} else {
  console.log("Minor");
}

for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}

while (count > 0) {
  count--;
}
```

---

## 📝 Работа со строками

```js
let text = "JavaScript";
text.length;                // 10
text.toUpperCase();         // "JAVASCRIPT"
text.includes("Script");    // true
text.replace("Java", "Type"); // "TypeScript"
```

---

## 📚 Массивы

```js
let fruits = ["🍎", "🍌", "🍍"];
fruits.push("🍓");             // добавить
fruits.pop();                  // удалить последний

fruits.map(f => f + "!");
fruits.filter(f => f !== "🍌");
fruits.find(f => f === "🍍");
```

---

## 🏷️ Объекты

```js
const user = {
  name: "Alice",
  age: 25
};

user.name;                   // "Alice"
user["age"];                 // 25

Object.keys(user);           // ["name", "age"]
Object.values(user);         // ["Alice", 25]
```

---

## 🚀 ES6+ Фишки

```js
// Деструктуризация
const { name, age } = user;

// Шаблонные строки
let msg = `Hello, ${name}! You are ${age} years old.`;

// Spread-оператор
let newArr = [...arr, 4, 5];
let newObj = { ...user, city: "Berlin" };

// Модули
// import { sum } from './math.js';
```

---

## 🌐 DOM & События

```js
// Поиск элемента
const btn = document.querySelector('#myBtn');

// Изменение текста
btn.textContent = "Click me!";

// События
btn.addEventListener('click', () => {
  alert('Clicked!');
});

// Получение/установка атрибута
btn.setAttribute('disabled', true);
```

---

## 🕸️ Запросы (fetch)

```js
fetch('/api/data')
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

---

## ⏳ Async/Await

```js
async function loadData() {
  try {
    const res = await fetch('/api/data');
    const data = await res.json();
    console.log(data);
  } catch (e) {
    console.error(e);
  }
}
```

---

## 🧹 Полезные утилиты

```js
// Проверить число
Number.isNaN(val);

// Клонировать объект
const copy = JSON.parse(JSON.stringify(obj));

// Генерировать случайное число
Math.floor(Math.random() * 100);

// Проверить массив
Array.isArray(arr);
```

---

## 🛠️ Советы

- Используйте `const` по умолчанию, `let` — если нужна переустновка значения.
- Не используйте `var`!
- Следите за областью видимости.
- Проверяйте работу кода в браузере.

---

*Сделано с любовью к JavaScript 💛*