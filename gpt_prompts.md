# 🌈 Промпты для GPT


## 📦 Задание для получения базового html+css из png макета

```text
1. Я дам тебе png макет веб-страницы - напиши адаптивную, семантичную HTML5, кроссбраузерную html+css верстку.
2. Используй Flex логику.
3. Мне нужна только общая структура блоков html с текстом и минимальный css чтобы сэкономить время на верстку.
Свойства width, height, margin, padding, font-size и так далее возьми пропорционально ширине контейнера, который я дам ниже.
По возможности все размеры указывай в px.
Детально остальной css я дабавлю руками. 
От тебя мне нужен своего рода эскиз страницы, но абсолютно все тексты из макета ты должен вставить в код.

4. Для background-image используй класс ibg:
css:
.ibg {background-position: center;background-size: cover;background-repeat: no-repeat;position: relative;}.ibg img {width: 0;height: 0;position: absolute;top: 0;left: 0;opacity: 0;visibility: hidden;}

html:
 <section class="hero ibg"><img src="./assets/img/hero_bg.webp" alt="Фон"></section>

js:
function ibg(){const t=document.querySelectorAll(".ibg");t.forEach(e=>{const r=e.querySelector("picture");if(!r)return;let c="";r.querySelectorAll("source").forEach(t=>{t.media&&window.matchMedia(t.media).matches&&(c=t.srcset)}),c||(c=r.querySelector("img")?.getAttribute("src")),c&&(e.style.backgroundImage=`url(${c}`)})}ibg(),window.addEventListener("resize",ibg);

5. Используй Block-Element-Modifier.
6. Изображения тут: "./assets/img/".
   Js тут: "./assets/js/script.js".
   Стили тут: "./assets/css/style.css".
7. В начало style.css добавь шрифт:
@import url('https://fonts.googleapis.com/css2?family=Onest:wght@100..900&display=swap');
и обнуляющие стили:
* {padding: 0;margin: 0;border: 0;}*, *:before, *:after {-moz-box-sizing: border-box;-webkit-box-sizing: border-box;box-sizing: border-box;}:focus, :active {outline: none;}a:focus, a:active {outline: none;}nav, footer, header, aside {display: block;}html, body {height: 100%;text-align: justify;width: 100%;font-size: 100%;line-height: 1;font-size: 14px;-ms-text-size-adjust: 100%;-moz-text-size-adjust: 100%;-webkit-text-size-adjust: 100%;}input, button, textarea {font-family: inherit;}input::-ms-clear {display: none;}button {cursor: pointer;}button::-moz-focus-inner {padding: 0;border: 0;}a, a:visited {text-decoration: none;}a:hover {text-decoration: none;}ul li {list-style: none;}img {vertical-align: top;}h1, h2, h3, h4, h5, h6 {font-size: inherit;font-weight: 400;}*, *:hover, *:before, *:after, *:not(:hover), {transition: 1s;}
8. Ширина страницы 1920px, контейнера 1430px.
На странице будет 3 блока, вот примерная структура, используй мои идентификаторы как базовые:

<section class="hero">
	<div class="container">...</div>
</section>
<section class="content">
	<div class="container">...</div>
</section>
<section class="articles">
	<div class="container">
		<article>...</article>
		<article>...</article>
		<article>...</article>
	</div>
</section>

9. Вот структура блока breadcrumbs:

<div class="breadcrumbs">
	<div id="breadcrumbs">
		<span>
			<span><a href="https://site.ua">Home</a></span> / 
			<span><a href="https://site.ua/cat">Category</a></span> / 
			<span class="breadcrumb_last">Page</span>
		</span>
	</div>
</div>
```
