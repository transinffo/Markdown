# 🌟 jQuery Cheat Sheet for Front-End Developers

## ✅ Функция анализа img, если полезное изображение только в центре, а по краям белые поля, то добавляем класс vertic, иначе horizont:
```js
 $(document).ready(function() {
    $('img').each(function() {
        const img = this;

        // Ждем загрузки изображения, чтобы получить данные
        if (img.complete) {
            analyzeImageOrientation(img);
        } else {
            $(img).on('load', function() {
                analyzeImageOrientation(img);
            });
        }
    });
});

function analyzeImageOrientation(imgElement) {
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d', { willReadFrequently: true });
    
    canvas.width = imgElement.naturalWidth;
    canvas.height = imgElement.naturalHeight;
    ctx.drawImage(imgElement, 0, 0);

    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;

    let minX = canvas.width, minY = canvas.height, maxX = 0, maxY = 0;
    let foundContent = false;

    // Проходим по пикселям (шаг 4 для ускорения)
    for (let y = 0; y < canvas.height; y += 4) {
        for (let x = 0; x < canvas.width; x += 4) {
            const index = (y * canvas.width + x) * 4;
            const r = data[index];
            const g = data[index + 1];
            const b = data[index + 2];
            const alpha = data[index + 3];

            // Условие "полезного контента": 
            // Пиксель не прозрачный и не чисто белый (можно настроить под ваш фон)
            if (alpha > 10 && !(r > 250 && g > 250 && b > 250)) {
                if (x < minX) minX = x;
                if (x > maxX) maxX = x;
                if (y < minY) minY = y;
                if (y > maxY) maxY = y;
                foundContent = true;
            }
        }
    }

    if (foundContent) {
        const contentWidth = maxX - minX;
        const contentHeight = maxY - minY;

        // Сравниваем размеры именно полезной области
        if (contentWidth >= contentHeight) {
            $(imgElement).addClass('horizont');
        } else {
            $(imgElement).addClass('vertic');
        }
    }
}
```
---

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

