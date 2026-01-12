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
<section class="hero ibg">
    <picture>
      <source srcset="./assets/img/hero_bg_mob.webp" media="(max-width: 519px)">
      <img src="./assets/img/hero_bg.webp" alt="Фон hero-секции">
    </picture>
</section>
 <!-- или -->
 <section class="hero ibg">
 	<img src="./assets/img/hero_bg.webp" alt="Фон hero-секции">
 </section>
```

```js
function ibg() {
	const ibgElements = document.querySelectorAll('.ibg');

	ibgElements.forEach(el => {
		const picture = el.querySelector('picture');
		if (!picture) return;

		let imgSrc = '';

		// если есть активный source
		const sources = picture.querySelectorAll('source');
		sources.forEach(source => {
			if (source.media && window.matchMedia(source.media).matches) {
				imgSrc = source.srcset;
			}
		});

		// fallback на img
		if (!imgSrc) {
			const img = picture.querySelector('img');
			if (img) imgSrc = img.getAttribute('src');
		}

		if (imgSrc) {
			el.style.backgroundImage = `url(${imgSrc})`;
		}
	});
}

ibg();
window.addEventListener('resize', ibg);

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
