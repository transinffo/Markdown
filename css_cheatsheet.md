# 🎨 CSS Cheat Sheet

---

## 🏷️ Прокрутка внутреннего текста у любого элемента при ховере

```html
<button type="submit" class="main_btn"><span>Применить</span></button>
<a href="#" class="main_btn"><span>Применить</span></a>
```

```css
.main_btn{
  background: coral;
  color: #fff;
  padding: 6px 20px;
  border-radius: 6px;
  display: inline-block;
  text-decoration: none;
  text-transform: none;
  border: none;
  display: inline-block;
  text-align: center;
  white-space: nowrap;
  overflow: hidden;
  position: relative;
}

.main_btn span {
  display: block;
}

@keyframes scrollDown {
  0% { transform: translateY(0); }
  50% { transform: translateY(100%); }
  50.01% { transform: translateY(-100%); } 
  100% { transform: translateY(0); } 
}

/* При ховере запускаем анимацию */
.main_btn:hover span {
  animation-name: scrollDown;
  animation-duration: 0.5s; 
  animation-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  animation-iteration-count: 1;
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

