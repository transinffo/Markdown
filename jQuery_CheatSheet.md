# 🌟 jQuery Cheat Sheet for Front-End Developers

## ✅ Базовое подключение

```html
<!-- CDN-подключение -->
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

## 🎬 События

```js
// Клик
$('#btn').click(function() {
  alert('Clicked!');
});

// Наведение мыши
$('.item').hover(
  function() { $(this).addClass('hover'); },
  function() { $(this).removeClass('hover'); }
);

// Делегирование событий
$('ul').on('click', 'li', function() {
  $(this).toggleClass('selected');
});
```

---

## ✏️ Манипуляции с DOM

```js
// Изменить текст
$('#title').text('Новый заголовок');

// Изменить HTML
$('#block').html('<b>Bold!</b>');

// Изменить значение input
$('input[name="user"]').val('John');

// Добавить/удалить класс
$('.box').addClass('active');
$('.box').removeClass('active');
$('.box').toggleClass('hidden');

// Показать/скрыть
$('#modal').show();
$('#modal').hide();
$('#modal').toggle();
```

---

## 🧩 Работа с атрибутами и стилями

```js
// Атрибуты
$('img').attr('src', 'logo.png');
$('a').removeAttr('target');

// Стили
$('.btn').css('background', 'orange');
$('.btn').css({ color: 'white', fontWeight: 'bold' });
```

---

## 📦 Массивы & каждый элемент

```js
// each
$('li').each(function(idx, el) {
  console.log(idx, $(el).text());
});
```

---

## 🔄 Анимации

```js
$('#box').fadeIn();
$('#box').fadeOut();
$('#box').slideUp();
$('#box').slideDown();

// Пользовательская анимация
$('#box').animate({ left: '100px', opacity: 0.5 }, 500);
```

---

## 🌐 AJAX-запросы

```js
$.get('/api/data', function(data) {
  console.log(data);
});

$.post('/api/user', { name: 'Alice' }, function(resp) {
  alert('Сохранено!');
});

// Общий AJAX
$.ajax({
  url: '/api/data',
  method: 'GET',
  dataType: 'json',
  success: function(res) { console.log(res); },
  error: function(e) { alert('Ошибка!'); }
});
```

---

## 🛠️ Полезные утилиты

```js
// Проверить наличие класса
if ($('.item').hasClass('selected')) { ... }

// Скрыть все элементы
$('.alert').hide();

// Получить значение
let val = $('#input').val();

// Получить/установить data-атрибут
$('.el').data('key', 123);
let v = $('.el').data('key');
```

---

## ⚡ Советы

- Всегда вызывайте jQuery-код внутри `$(document).ready()`
```js
$(function() {
  // Ваш код здесь
});
```
- Используйте делегирование событий для динамических элементов.
- jQuery отлично работает в связке с Bootstrap.

---

*Сделано с любовью к jQuery 💙*