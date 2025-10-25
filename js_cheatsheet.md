# 🌈 JavaScript Cheat Sheet

## 📦 Класс ibg

```css
.ibg {
  background-position: center;
  background-size: cover;
  background-repeat: no-repeat;
  position: relative;
}
.ibg img {
  display: none; 
}
```

```html
<div class="ibg">
  <img src="image/banner-desktop.webp" alt="фон для компьютера" class="desk">
  <img src="image/banner-mobile.webp" alt="фон для телефона" class="mob">
</div>
```

```js
function ibg() {
  const ibgBlocks = document.querySelectorAll('.ibg');
  ibgBlocks.forEach(block => {
    const imgDesktop = block.querySelector('img.desk');
    const imgMobile = block.querySelector('img.mob');
    const isMobile = window.innerWidth <= 768;
    const src = isMobile && imgMobile ? imgMobile.src : imgDesktop ? imgDesktop.src : null;

    if (src) {
      block.style.backgroundImage = `url('${src}')`;
    }
  });
}
ibg();
window.addEventListener('resize', ibg);
```



## 📦 Основы синтаксиса

---

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
