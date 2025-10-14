# 🌟 jQuery Cheat Sheet for Front-End Developers


## ✅ 2 варианта языка в содержимом тега в зависимости от значения атрибута lang тега html:
### Используем так:
```html
<h2 class="checkout-title" data-lang-uk="ukkk" data-lang-ru="ruuu"></h2>
<input type="text" value="1" placeholder="украинский текст|русский текст">
```
```html
function applyLanguageTranslation() {
    const lang = $('html').attr('lang');
    const langIndex = (lang === 'uk') ? 0 : 1;
    
    const targetAttr = (lang === 'uk') ? 'data-lang-uk' : 'data-lang-ru'; 

    $('[data-lang-uk]').each(function() {
        const translation = $(this).attr(targetAttr);
        if (translation) {
            $(this).text(translation);
        }
    });

    $('[placeholder]').each(function() {
        const placeholderText = $(this).attr('placeholder');
        
        if (placeholderText && placeholderText.includes('|')) {
            const translations = placeholderText.split('|');
            
            if (translations.length > langIndex) {
                $(this).attr('placeholder', translations[langIndex].trim());
            }
        }
    });
}
// Запускаем при загрузке страницы для статичных элементов
$(document).ready(function(){
    applyLanguageTranslation();
});

//если отрисовываем динамический контент, то добавляем в функцию рендера  applyLanguageTranslation();
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

