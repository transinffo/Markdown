# 🌟 jQuery Cheat Sheet for Front-End Developers


## ✅ 2 варианта языка в содержимом тега в зависимости от значения атрибута lang тега html:
### Используем так:
```html
<h2 class="checkout-title" data-lang-uk="Ваш кошик" data-lang-ru="Ваша корзина"></h2>
```
```html
$(document).ready(function(){
  const lang = $('html').attr('lang');
  $('[data-lang-uk]').each(function() {
    if (lang === 'uk') {
      $(this).text($(this).data('lang-uk'));
    } else if (lang === 'ru-RU') {
      $(this).text($(this).data('lang-ru'));
    }
  });
});
```

---

## ✅ Координаты указателя в title

```html
$(document).ready(function(){
    $(document).on('mousemove', function(e){
      document.title = 'x: ' + e.pageX + ' y: ' + e.pageY;
    });
});
```

---

## ✅ Базовое подключение

```html
<script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>
```

---

## 🔎 Селекторы

```js
// По ID
$('#myId')

// По классу
$('.myClass')

// По тегу
$('div')

// Комбинированно
$('ul.menu li.active')
```

---

