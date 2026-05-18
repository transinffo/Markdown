# 🌈 Промпты для GPT


## 📦 Промпт для плагина перевода сложного контента в WP.

```text
1. Напиши код уровня production grade для WP плагина "Manual content translator".

2. Пример конфигурации сайта: 

Wp 6.9 + woocommerce 10.7.0 + polylang 3.8.3. 
Настройки языков для polylang:
Українська: локаль - uk; код - uk; язык по умолчанию.
Русский: локаль - ru_RU; код - ru.

Языки указаны для примера, тебе нужно получить список языков из polylang для работы нашего плагина.

3. Для работы плагина используй:
translate.googleapis.com (бесплатный, неофициальный) либо  библиотеки-обёртки (PHP/JS), которые используют тот же неофициальный endpoint под капотом.

4. Плагин добавляет к input и textarea на страницах WP иконку в правом верхнем углу поля с выпадающим списком языков из polylang. В зависимости от выбранного языка, плагин отправляет содержимое поля на перевод в google. Полученный переведенный контент, вставляется в поле.

Иконка - в виде земного шара с меридианами, во время процесса перевода шарик крутится, показывая обработку данных.
Перед переводом шарик голубой, после удачного окончания - зеленый. В случае ошибки, по наведению на шарик - выскакивает tooltip с текстом ошибки. В случае ошибки, исходный контент не трогаем и оставляем как есть.

5. По возможности, язык исходного контента, скрипт должен определить автоматически.

Ниже примеры полей, с которыми прийдется работать.

textarea woo product, posts, pages примеры:
<!-- полное описание товара -->
<textarea class="wp-editor-area" autocomplete="off" cols="40" name="content" id="content">textarea content</textarea>

<!-- короткое описание товара -->
<textarea class="wp-editor-area" rows="20" autocomplete="off" cols="40" name="excerpt" id="excerpt">textarea content</textarea>

<!-- кастомное поле -->
<textarea id="acf-editor-60" class="wp-editor-area" name="acf[field_5fba6d37754fb]"></textarea>

<!-- таб характеристика товара (доп. функция темы) -->
<textarea class="cmb2_textarea" name="custom_tab1" id="custom_tab1" cols="60" rows="10" data-hash="ojfv5h7acdc0">textarea content</textarea>


inputs woo product, примеры:
<!-- заголовок товара -->
<input type="text" name="post_title" size="30" value="DJI Phantom 4" id="title" spellcheck="true" autocomplete="off">

<!-- кастомное поле товара -->
<input type="text" id="acf-field_5fba6d1a754f8" name="acf[field_5fba6d1a754f8]">

<!-- кастомные поля темы -->
<input type="text" class="regular-text" name="custom_tab1_title" id="custom_tab1_title" value="Характеристики / Комплектація" data-hash="5qd4piqd7bf0">


6. Контент полей, особенно textarea может быть сложным и может содержать обычный текст, нативные шорткоды WP, html, одиночные и парные теги WPbakery Page Builder, Elementor и других текстовых редакторов WP, а также их комбинации.

Общее правило перевода - переводим только содержимое тегов. Сами теги, их атрибуты, значения атрибутов, классы, идентификаторы, url, названия файлов на кириллице, css правила, стили - не переводим.

Исключение составляют атрибуты "title" и "alt" тегов. Например для этих тегов:

[vc_tta_section title="Настройка пульта" tab_id="1469138890801-2dbd2fcf-b3db"]
[vc_video link="https://www.youtube.com/watch?v=JJPSSqMQajA" title="DJI Phantom 4 - Видео обзор"]
<img class="aligncenter wp-image-22772 size-full" src="https://dronestore.com.ua/wp-content/uploads/2025/10/фото-дрона_1.jpg" alt="Обзор дрона dji neo 2" width="730" height="632" />

Нужно перевести значения атрибутов:
title="Настройка пульта"
title="DJI Phantom 4 - Видео обзор"
alt="Обзор дрона dji neo 2"

6.1 Комментари можно не переводить, например:
<!-- обычный шорткод для cf7 -->

6.2 Перевод должен быть устойчив к немного нестандартным стилям вида:
[vc_column css=".vc_custom_1493407566663{padding-top: 100px !important;}"]

Нестандартность в фигурных скобках внутри квадратных.


6.3 Перевод должен быть устойчив к  html со спецсимволами (типа "&nbsp;" или "&#x2705;") и графическими элементами типа "•":

"<h3>Захват для DJI Ronin комплект:</h3>
• С-образные ручки - 2 шт.
• монтажные винты - 2 шт.
• кейс сумка - 1 шт.

&nbsp;

<h3>&#x2705; <strong>Кому що підійде</strong></h3>"


6.4 Также обрати внимание на Tailwind-скобки ("py-[0.2rem]"), чтобы плагин правильно их обрабатывал, пример:
"<div id="bxr-detail-block-wrap" class="row tb20 py-[0.2rem]">"

6.5 Нужно сохранить внешний вид исходного контента: пробелы, отступы, переводы строк.

7. В одном из своих проектов я использовал apps script для перевода ячеек гугл таблицы, мне понравилось как он работает, можешь его проанализировать и применить методы и подходы этого скрипта для нашего плагина. Код скрипта можешь посмотреть по ссылке:
https://raw.githubusercontent.com/transinffo/google-reviews/refs/heads/main/apps_script.js

8. Страница с настройками плагина не нужна.

9. Не важно какой текстовый редактор использует пользователь (например: gutenberg, wpbakery page builder, elementor) в админке WP, перевод будет всегда в режиме "classic editor".

10. Иконка перевода должна появиться на страницах с любым типом post_type и taxonomy, например:
    страница поста post_type=post
    страница обычная post_type=page
    страница woo товара post_type=product
    категория поста taxonomy=category
    категория woo товара taxonomy=product_cat
```
