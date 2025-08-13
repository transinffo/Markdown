# 🎨 CSS Cheat Sheet for Front-End & WordPress Developers

---

## ✨ Основы синтаксиса

```css
/* Комментарий */
.selector {
  property: value;
}

/* Пример */
body {
  background: #f6f7f8;
  color: #333;
  font-family: 'Inter', Arial, sans-serif;
}
```

---

## 🏷️ Селекторы

```css
/* По тегу */
div { ... }

/* По классу */
.box { ... }

/* По ID */
#main { ... }

/* Вложенные */
nav ul li a { ... }

/* Группировка */
h1, h2, h3 { ... }

/* Универсальный */
* { box-sizing: border-box; }

/* Псевдокласс */
a:hover { color: orange; }
input:focus { border: 1px solid blue; }
li:first-child, li:last-child { ... }
:nth-child(2n) { background: #eee; }

/* Псевдоэлемент */
p::after { content: '★'; }
h1::before { content: '→ '; }

/* Атрибутный */
input[type="checkbox"] { ... }
a[target="_blank"] { ... }
```

---

## 📐 Блочная модель

```css
/* Размеры */
width: 100px;
height: 32px;
max-width: 100%;
min-height: 40px;

/* Отступы */
margin: 0 auto;
margin-top: 20px;
padding: 10px 15px;

/* Границы */
border: 1px solid #ccc;
border-radius: 8px;
box-shadow: 0 2px 8px rgba(0,0,0,0.12);
```

---

## 🎯 Позиционирование

```css
position: static;
position: relative;
position: absolute; /* top/right/bottom/left */
position: fixed;
position: sticky;

top: 0;
left: 50px;
z-index: 10;
float: right;
clear: both;
```

---

## 🧩 Flexbox

```css
.container {
  display: flex;
  flex-direction: row; /* column */
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 16px;
}

.item {
  flex: 1 1 200px;
  align-self: stretch;
  order: 2;
}
```

---

## 🟩 Grid

```css
.grid {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  grid-template-rows: auto 100px;
  gap: 12px;
  grid-column: 2 / 4;
  grid-row: 1 / span 2;
}

.item {
  grid-area: 2 / 1 / 3 / 3;
}
```

---

## 🖌️ Цвета и фон

```css
color: #1e90ff;
background: #fff;
background-color: #f5f5f5;
background-image: url('img/bg.jpg');
background-repeat: no-repeat;
background-position: center;
background-size: cover;
opacity: 0.7;
```

---

## 📝 Шрифты и текст

```css
font-size: 1.2rem;
font-weight: bold;
font-family: 'Montserrat', sans-serif;
line-height: 1.4;
text-align: center;
text-transform: uppercase;
letter-spacing: 2px;
word-break: break-all;
white-space: nowrap;
text-decoration: underline;
text-shadow: 1px 1px 3px #999;
```

---

## 🖼️ Изображения и медиа

```css
img {
  max-width: 100%;
  height: auto;
  border-radius: 8px;
  object-fit: cover;
  box-shadow: 0 2px 10px #ddd;
}

video, iframe {
  width: 100%;
  aspect-ratio: 16/9;
}
```

---

## 🕹️ Анимации и переходы

```css
/* Переходы */
.button {
  transition: background 0.3s, transform 0.2s;
}
.button:hover {
  background: orange;
  transform: scale(1.05);
}

/* Анимации */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
.modal {
  animation: fadeIn 0.6s ease;
}
```

---

## 🛠️ Responsive & адаптив

```css
/* Медиа-запросы */
@media (max-width: 768px) {
  .menu { flex-direction: column; }
  .sidebar { display: none; }
}

@media (min-width: 1200px) {
  .container { max-width: 1100px; }
}

/* Responsive Units */
width: 100vw;
height: 50vh;
font-size: 2em;
padding: 2rem;
```

---

## 🧩 Utility и Reset

```css
/* Сброс стилей */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* Скрыть элемент */
.hidden { display: none; }
.invisible { visibility: hidden; }

/* Скрыть скроллбар */
::-webkit-scrollbar { display: none; }

/* Убрать outline */
:focus { outline: none; }
```

---

## 🌟 Полезные эффекты и трюки

```css
/* Кнопка с градиентом */
.btn {
  background: linear-gradient(90deg, #ff8008, #ffc837);
  color: #fff;
  border: none;
  box-shadow: 0 2px 8px #ffc83755;
}

/* Картинка с overlay */
.image-wrapper {
  position: relative;
}
.image-wrapper::after {
  content: '';
  position: absolute;
  inset: 0;
  background: rgba(0,0,0,0.3);
}

/* Адаптивное изображение с aspect-ratio */
.img-ratio {
  aspect-ratio: 16/9;
  width: 100%;
  object-fit: cover;
}

/* Кнопка с ripple */
.button:active {
  box-shadow: 0 0 12px #ff8008;
}

/* Плавный скролл */
html {
  scroll-behavior: smooth;
}
```

---

## ⚡ Советы

- Используйте переменные CSS для цветов и размеров: `--main-color: #1e90ff;`
- Предпочитайте flex и grid для layout.
- Для адаптива — медиа-запросы, относительные единицы (`em`, `%`, `vw`, `rem`).
- Не бойтесь использовать кастомные свойства и псевдоэлементы.
- Проверяйте кроссбраузерность и accessibility.
- Минимизируйте и группируйте стили для производительности.
- Используйте BEM-нейминг для классов.
- Старайтесь не использовать `!important` без необходимости.
- Для WordPress — подключайте стили через `wp_enqueue_style`.
- Для темных тем — используйте media query `prefers-color-scheme: dark`.

---

*Шпаргалка CSS для Full Stack · WP · Front-end · Адаптив 🚀*