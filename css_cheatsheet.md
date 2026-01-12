# 🎨 CSS Cheat Sheet

## 🏷️ Затенение с любой стороны (в примере белым слева и до середины)
```css
/* Базовый класс */
.overlay {
  position: relative;
}

/* Оверлей */
.overlay::before {
  content: "";
  position: absolute;
  inset: 0;
  pointer-events: none;

  /* НАСТРОЙКИ ПО УМОЛЧАНИЮ */
  --overlay-color: 255, 255, 255; /* RGB */
  --overlay-from: right;          /* left | right | top | bottom */
  --overlay-strong: 0.55;        /* сила в начале */
  --overlay-weak: 0;             /* сила в конце */
  --overlay-stop: 55%;           /* где заканчивается */

  background: linear-gradient(
    to var(--overlay-from),
    rgba(var(--overlay-color), var(--overlay-strong)) 0%,
    rgba(var(--overlay-color), var(--overlay-weak)) var(--overlay-stop)
  );
}
```

## 🏷️ Плавное расстворение изображения снизу
```css
.fade_bottom{
  mask-image: linear-gradient(
        to top,
        transparent 0%,
        rgba(0,0,0,0.3) 20%,
        rgba(0,0,0,0.7) 40%,
        black 60%
    );
    mask-size: 100% 100%;
    mask-repeat: no-repeat;
}
```
    

## 🏷️ Адаптивная таблица
```css
/*Адаптивная таблица*/

@media  (max-width: 500px){
    table {
        width: 100%;
        display: block; 
        margin-bottom: 20px;           
        overflow-x: auto;            
        -webkit-overflow-scrolling: touch;
        white-space: break-word;
         /* визуальное отделение скроллбара от контента */
        padding-bottom: 12px;            /* место под скролл, как подложка */
        border-bottom: 3px solid #e5e5e5; /* чёткая линия-разделитель */
        box-shadow: inset 0 -6px 6px -6px rgba(0,0,0,0.15); /* лёгкая тень */
    }
}

/* ====== WebKit scrollbar (Chrome/Safari/Edge/Opera) ====== */
table::-webkit-scrollbar {
    height: 20px;                    /* делаем его более заметным */
}

table::-webkit-scrollbar-track {
    background: #eaeaea;             /* светлая дорожка */
    border-radius: 5px;
    border: 1px solid #d5d5d5;       /* отделяет дорожку от фона */
}

table::-webkit-scrollbar-thumb {
    background: #666;             /* сам ползунок — контраст виден всегда */
    border-radius: 5px;
}

table::-webkit-scrollbar-thumb:hover {
    background: #404040;
}

/* ====== Firefox ====== */
table {
    scrollbar-width: thin;
    scrollbar-color: #b2b2b2 #eaeaea; /* thumb / track */
}
/*Адаптивная таблица*/
```

---


## 🏷️ Плавное изменение

```css
li {
    transition: all ease 0.3s;
}
li:hover{
  transform: scale(.95);
}
```

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

