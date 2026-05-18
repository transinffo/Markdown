# 🌈 JavaScript Cheat Sheet

## 📦 Программно кликаем по переводу заголовка и контента поста

```js
// ==================== НАСТРОЙКИ (КОНФИГ) ====================
const TARGET_LANG = 'ru'; // Код целевого языка ('uk' или 'ru')
const DELAY = 5000;       // Единая задержка для всего (клики, ожидания) в мс

// Шаблон URL. Обязательно оставляй {post_id} там, где должен быть ID страницы
const URL_TEMPLATE = 'https://test.dronestore.com.ua/wp-admin/post.php?post={post_id}&action=edit';

// Твой массив ID постов
const POST_IDS = [
    25121, 25122, 25118, 25119, 25120, 25115, 25116, 25117, 25114, 
    25113, 25111, 25112, 25110, 25108, 25109, 25107, 25105, 25106, 25103, 
    25104, 25102, 25100, 25101, 25098, 25099, 25097, 25096, 25094, 25095, 25093
];
// ============================================================

// Сюда собираем статистику
const result_array = [];

// Универсальная функция задержки
const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));

// Функция-помощник: ждет появления класса mct-state-success у элемента
async function waitForSuccessClass(element, postId, blockName) {
    return new Promise((resolve) => {
        if (element.className.includes('mct-state-success')) {
            resolve(true);
            return;
        }

        const observer = new MutationObserver((mutationsList) => {
            for (const mutation of mutationsList) {
                if (mutation.type === 'attributes' && mutation.attributeName === 'class') {
                    if (element.className.includes('mct-state-success')) {
                        observer.disconnect();
                        console.log(`[ID ${postId}][Сделано] ${blockName} переведен успешно.`);
                        resolve(true);
                    }
                }
            }
        });

        observer.observe(element, { attributes: true, attributeFilter: ['class'] });
    });
}

// Главный управляющий раннер
async function runMassAutomation() {
    console.log(`%c[СТАРТ] Начинаем автоматическую обработку ${POST_IDS.length} постов...`, 'color: #0073aa; font-weight: bold; font-size: 14px;');

    const iframe = document.createElement('iframe');
    iframe.style.position = 'fixed';
    iframe.style.top = '0';
    iframe.style.left = '0';
    iframe.style.width = '100vw';
    iframe.style.height = '100vh';
    iframe.style.zIndex = '999999';
    iframe.style.border = 'none';
    iframe.style.background = '#fff';
    document.body.appendChild(iframe);

    for (let i = 0; i < POST_IDS.length; i++) {
        const postId = POST_IDS[i];
        console.log(`%c\n--- [Пост ${i + 1} из ${POST_IDS.length}] Работаем с ID: ${postId} ---`, 'color: #d54e21; font-weight: bold;');

        const logEntry = await processSinglePost(iframe, postId);
        result_array.push(logEntry);
    }

    // Завершение работы
    iframe.remove();
    console.log('%c\n[УСПЕХ] Робот закончил работу! Итоговый отчет ниже:', 'color: #46b450; font-weight: bold; font-size: 14px;');
    
    // Вывод красивой таблицы результатов
    console.table(result_array);
}

// Функция обработки одного конкретного поста
async function processSinglePost(iframe, postId) {
    const status = {
        post_id: postId,
        page_download: 'failed',
        title_translate: 'skipped',
        content_translate: 'skipped',
        page_save: 'skipped'
    };

    return new Promise((resolve) => {
        // Динамически подставляем ID в шаблон адреса страницы
        iframe.src = URL_TEMPLATE.replace('{post_id}', postId);

        iframe.onload = async () => {
            console.log(`[ID ${postId}] Страница загружена. Ожидаем инициализацию...`);
            status.page_download = 'ok';
            await delay(DELAY);

            const iframeDoc = iframe.contentDocument || iframe.contentWindow.document;

            // 1. ПЕРЕВОД ЗАГОЛОВКА
            const titleGlobeBtn = iframeDoc.querySelector('#titlewrap .mct-wrapper button.mct-globe-btn');
            if (!titleGlobeBtn) {
                console.error(`[ID ${postId}][Ошибка] Глобус ЗАГОЛОВКА не найден.`);
                resolve(status); return;
            }

            console.log(`[ID ${postId}][Заголовок] Клик по глобусу.`);
            titleGlobeBtn.click();
            await delay(DELAY);

            const titleLangBtn = iframeDoc.querySelector(`#titlewrap .mct-wrapper .mct-dropdown button[data-code="${TARGET_LANG}"]`);
            if (!titleLangBtn) {
                console.error(`[ID ${postId}][Ошибка] Язык заголовка не найден.`);
                resolve(status); return;
            }

            console.log(`[ID ${postId}][Заголовок] Клик по языку.`);
            titleLangBtn.click();
            
            await waitForSuccessClass(titleGlobeBtn, postId, 'Заголовок');
            status.title_translate = 'ok';
            await delay(DELAY);

            // 2. ПЕРЕВОД КОНТЕНТА
            const contentGlobeBtn = iframeDoc.querySelector('#content + .mct-wrapper button.mct-globe-btn');
            if (!contentGlobeBtn) {
                console.error(`[ID ${postId}][Ошибка] Глобус КОНТЕНТА не найден.`);
                resolve(status); return;
            }

            console.log(`[ID ${postId}][Контент] Клик по глобусу.`);
            contentGlobeBtn.click();
            await delay(DELAY);

            const contentLangBtn = iframeDoc.querySelector(`#content + .mct-wrapper .mct-dropdown button[data-code="${TARGET_LANG}"]`);
            if (!contentLangBtn) {
                console.error(`[ID ${postId}][Ошибка] Язык контента не найден.`);
                resolve(status); return;
            }

            console.log(`[ID ${postId}][Контент] Клик по языку.`);
            contentLangBtn.click();

            await waitForSuccessClass(contentGlobeBtn, postId, 'Контент');
            status.content_translate = 'ok';
            await delay(DELAY);

            // 3. СОХРАНЕНИЕ
            const publishBtn = iframeDoc.querySelector('input#publish');
            if (!publishBtn) {
                console.error(`[ID ${postId}][Ошибка] Кнопка "Обновить" не найдена.`);
                resolve(status); return;
            }

            console.log(`[ID ${postId}][Сохранение] Клик по "Обновить".`);
            publishBtn.click();
            
            // Задержка DELAY * 2 по ТЗ перед закрытием / переходом
            await delay(DELAY * 2); 
            status.page_save = 'ok';
            console.log(`[ID ${postId}][ОК] Изменения сохранены.`);

            resolve(status);
        };
    });
}

// Запуск автоматизации
runMassAutomation();
```

## 📦 Программно кликаем по свойствам категории в WP и обновляем

```js
(async () => {
    // 1. Собираем все кнопки "Свойства"
    const buttons = Array.from(document.querySelectorAll('#the-list button.button-link.editinline'));
    console.log(`Найдено элементов: ${buttons.length}. Начинаю автоматизацию...`);

    for (let i = 0; i < buttons.length; i++) {
        const btn = buttons[i];
        console.log(`--- Обработка строки ${i + 1} из ${buttons.length} ---`);

        // Скроллим к элементу, чтобы видеть процесс
        btn.scrollIntoView({ behavior: 'smooth', block: 'center' });

        // Нажимаем "Свойства"
        btn.click();

        // Ждем 1.5 секунды, чтобы форма точно успела открыться в DOM
        await new Promise(resolve => setTimeout(resolve, 1500));

        // Ищем строку редактирования (она появляется под текущей или вместо неё)
        const editRow = document.querySelector('#the-list tr.inline-edit-row');
        
        if (editRow) {
            const saveButton = editRow.querySelector('button.save.button.button-primary');

            if (saveButton) {
                console.log('Нажимаю "Сохранить"...');
                saveButton.click();
            } else {
                console.warn('Кнопка "Сохранить" не найдена в этой строке.');
            }
        } else {
            console.warn('Форма редактирования не открылась.');
        }

        // Ждем 3 секунды перед переходом к следующей итерации
        console.log('Ожидание 3 секунды перед следующим элементом...');
        await new Promise(resolve => setTimeout(resolve, 3000));
    }

    console.log('✅ Все задачи выполнены!');
})();
```

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
