# 🎨 CSS Cheat Sheet

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

