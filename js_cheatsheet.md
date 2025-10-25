# 🌈 JavaScript Cheat Sheet

## 📦 Класс ibg

```html
<style>

	.ibg {
  background-position: center;
  background-size: cover;
  background-repeat: no-repeat;
  position: relative;
}

.ibg img {
  display: none; 
}

</style>


<div class="ibg">
  <img src="image/banner-desktop.webp" alt="фон для компьютера" class="desktop">
  <img src="image/banner-mobile.webp" alt="фон для телефона" class="mobile">
</div>


<script>
	function ibg() {
  const ibgBlocks = document.querySelectorAll('.ibg');
  ibgBlocks.forEach(block => {
    const imgDesktop = block.querySelector('img.desktop');
    const imgMobile = block.querySelector('img.mobile');
    const isMobile = window.innerWidth <= 768; // нужный порог можно менять
    const src = isMobile && imgMobile ? imgMobile.src : imgDesktop ? imgDesktop.src : null;

    if (src) {
      block.style.backgroundImage = `url('${src}')`;
    }
  });
}

ibg();
window.addEventListener('resize', ibg); // обновляем фон при изменении ширины
</script>
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
