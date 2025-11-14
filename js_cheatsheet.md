# 🌈 JavaScript Cheat Sheet


## 📦 Класс ibg

```css
.ibg{
background-position: center;
background-size: cover;
background-repeat: no-repeat;
position: relative;
}

.ibg img{
width: 0;
height: 0;
position: absolute;
top: 0;
left: 0;
opacity: 0;
visibility: hidden;
}
```

```html
<div class="ibg">
  <img src="image/banner-desktop.webp" alt="фон для компьютера">
</div>
```

```js
function ibg() {
	let ibg = document.querySelectorAll(".ibg");
	for(var i = 0; i < ibg.length; i++) {
		if(ibg[i].querySelector('img')) {
			ibg[i].style.backgroundImage = 'url(' + ibg[i].querySelector('img').getAttribute('src') + ')';
		}
	}
}
ibg();
```

## 📦 Класс ibg для 2 img

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
