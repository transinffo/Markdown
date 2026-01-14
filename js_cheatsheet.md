# 🌈 JavaScript Cheat Sheet

## 📦 Маска для телефона (input clas="phone")
```js
document.addEventListener('focusin', function (e) {
  if (!e.target.classList.contains('phone')) return;

  if (!e.target.value) {
    e.target.value = '+38 (0';
    setTimeout(() => {
      e.target.setSelectionRange(e.target.value.length, e.target.value.length);
    });
  }
});

document.addEventListener('input', function (e) {
  if (!e.target.classList.contains('phone')) return;

  let x = e.target.value.replace(/\D/g, '');

  if (!x.startsWith('380')) {
    x = '380' + x.replace(/^38?0?/, '');
  }

  x = x.slice(0, 12);

  let r = '+38 (';

  if (x.length > 2) r += x.slice(2, 5);
  if (x.length > 5) r += ') ' + x.slice(5, 8);
  if (x.length > 8) r += '-' + x.slice(8, 10);
  if (x.length > 10) r += '-' + x.slice(10, 12);

  e.target.value = r;
});
```

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

## 📦 ---

```css

```

```html

```

```js

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
