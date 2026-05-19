# 🌈 JavaScript Cheat Sheet

## 📦 Программно кликаем по переводу заголовка и контента поста

```js
// ==================== НАСТРОЙКИ (КОНФИГ) ====================
const TARGET_LANG = 'uk'; // Код целевого языка ('uk' или 'ru')
const DELAY = 1000;       // Единая задержка для всего (клики, ожидания) в мс
const MAX_ATTEMPTS = 3;   // Количество попыток загрузки страницы при сбое сети
const RETRY_DELAY = 15000; // Сколько ждать (в мс) перед повторной попыткой при ошибке

// Шаблон URL. Обязательно оставляй {post_id} там, где должен быть ID страницы

// ходим по страницам woo товара, постам и страницам
//const URL_TEMPLATE = 'https://test.dronestore.com.ua/wp-admin/post.php?post={post_id}&action=edit';
// const FIELDS_TO_TRANSLATE = [
//     { name: 'Заголовок', selector: '#title' },
//     { name: 'Контент', selector: '#content' },
//     { name: 'Краткое описание', selector: '#excerpt' },
//     { name: 'Кастомный таб заголовок', selector: '#custom_tab1_title' },
//     { name: 'Кастомный таб контент', selector: '#custom_tab1' }
// ];

// ходим по категориям woo товара
const URL_TEMPLATE = 'https://test.dronestore.com.ua/wp-admin/term.php?taxonomy=product_cat&tag_ID={post_id}&post_type=product';
const FIELDS_TO_TRANSLATE = [
    { name: 'Заголовок', selector: '#name' },
    { name: 'Описание', selector: '#description' }
];



// Твой массив ID постов
const POST_IDS = [
    169, 240, 189, 188, 237, 190, 199, 211, 170, 273, 191, 192, 193, 297, 195, 194, 95, 81, 80, 79, 64, 67, 7, 9, 8, 135, 18, 69, 10, 172, 17, 164, 166, 167, 68, 143
];
// ============================================================

// Ключи для сохранения состояния в браузере
const STORAGE_KEY_RESULTS = 'wp_automation_results';
const STORAGE_KEY_DONE_IDS = 'wp_automation_done_ids';

// Инициализация хранилища (достаем старый прогресс, если он есть)
const result_array = JSON.parse(localStorage.getItem(STORAGE_KEY_RESULTS)) || [];
const done_post_ids = new Set(JSON.parse(localStorage.getItem(STORAGE_KEY_DONE_IDS)) || []);

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
    const startTime = performance.now();

    // Фильтруем массив: оставляем только те ID, которые ЕЩЕ НЕ переведены
    const remainingIds = POST_IDS.filter(id => !done_post_ids.has(id));

    if (remainingIds.length === 0) {
        console.log('%c[ИНФО] Все посты из списка уже были успешно обработаны ранее! Сбрасываю кэш.', 'color: #cca300; font-weight: bold;');
        clearProgress();
        return;
    }

    if (done_post_ids.size > 0) {
        console.log(`%c[ПРОДОЛЖЕНИЕ] Найдена сохраненная сессия. Уже сделано: ${done_post_ids.size}. Осталось обработать: ${remainingIds.length} постов.`, 'color: #73aa00; font-weight: bold;');
    } else {
        console.log(`%c[СТАРТ] Начинаем автоматическую обработку ${POST_IDS.length} постов...`, 'color: #0073aa; font-weight: bold; font-size: 14px;');
    }

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

    // Идем только по оставшимся ID
    for (let i = 0; i < remainingIds.length; i++) {
        const postId = remainingIds[i];
        const overallIndex = done_post_ids.size + 1;
        console.log(`%c\n--- [Пост ${overallIndex} из ${POST_IDS.length}] Работаем с ID: ${postId} ---`, 'color: #d54e21; font-weight: bold;');

        let logEntry = null;
        let attempt = 1;
        let success = false;

        // Цикл повторных попыток на случай обрыва сети
        while (attempt <= MAX_ATTEMPTS && !success) {
            try {
                if (attempt > 1) {
                    console.log(`%c[ID ${postId}] Попытка №${attempt}... Ожидаем восстановления сети (${RETRY_DELAY / 1000} сек)`, 'color: #cca300;');
                    await delay(RETRY_DELAY);
                }
                
                logEntry = await processSinglePost(iframe, postId);
                
                // Если страница вообще не загрузилась (сеть легла до onload)
                if (logEntry.page_download === 'failed') {
                    throw new Error('Ошибка загрузки страницы (возможно, нет интернета)');
                }
                
                success = true; // Если дошли сюда без ошибок — всё ок
            } catch (error) {
                console.error(`[ID ${postId}][Сбой сети] Ошибка на попытке ${attempt}: ${error.message}`);
                attempt++;
            }
        }

        // Если все попытки провалились
        if (!success) {
            console.error(`%c[ID ${postId}][ПРОПУСК] Не удалось обработать пост после ${MAX_ATTEMPTS} попыток. Переходим к следующему.`, 'color: #dc3232; font-weight: bold;');
            logEntry = { post_id: postId, page_download: 'network_error_timeout' };
            FIELDS_TO_TRANSLATE.forEach(f => logEntry[f.name] = 'failed');
            logEntry.page_save = 'failed';
        }

        // Сохраняем шаг в кэш браузера, чтобы не потерять при перезагрузке страницы
        result_array.push(logEntry);
        done_post_ids.add(postId);
        
        localStorage.setItem(STORAGE_KEY_RESULTS, JSON.stringify(result_array));
        localStorage.setItem(STORAGE_KEY_DONE_IDS, JSON.stringify(Array.from(done_post_ids)));
    }

    // Завершение работы с DOM
    iframe.remove();
    
    const endTime = performance.now();
    const timeSpentMs = endTime - startTime;
    const minutes = Math.floor(timeSpentMs / 60000);
    const seconds = Math.floor((timeSpentMs % 60000) / 1000);

    console.log('%c\n[УСПЕХ] Робот закончил работу! Итоговый отчет ниже:', 'color: #46b450; font-weight: bold; font-size: 14px;');
    console.table(result_array);
    console.log(`%c⏱️ Время текущей сессии: ${minutes} мин. ${seconds} сек.`, 'color: #0073aa; font-weight: bold; font-size: 13px; background: #f0f6fb; padding: 8px; border-left: 4px solid #11a0d2;');

    // Очищаем кэш после успешного выполнения всего списка
    clearProgress();
}

// Функция очистки кэша прогресса
function clearProgress() {
    localStorage.removeItem(STORAGE_KEY_RESULTS);
    localStorage.removeItem(STORAGE_KEY_DONE_IDS);
    console.log('%c[КЭШ ПОДЧИЩЕН] Состояние сброшено, следующий запуск начнется заново.', 'color: #999; font-style: italic;');
}

// Функция обработки одного конкретного поста
async function processSinglePost(iframe, postId) {
    const status = { post_id: postId, page_download: 'failed' };
    FIELDS_TO_TRANSLATE.forEach(field => status[field.name] = 'skipped');
    status['page_save'] = 'skipped';

    return new Promise((resolve, reject) => {
        // Ставим таймаут на случай, если iframe зависнет при мёртвом интернете
        const networkTimeout = setTimeout(() => {
            resolve(status); 
        }, 30000); // 30 секунд на загрузку страницы, иначе считаем ошибкой

        iframe.src = URL_TEMPLATE.replace('{post_id}', postId);

        iframe.onload = async () => {
            clearTimeout(networkTimeout);
            console.log(`[ID ${postId}] Страница загружена. Ожидаем инициализацию...`);
            status.page_download = 'ok';
            await delay(DELAY);

            let iframeDoc;
            try {
                iframeDoc = iframe.contentDocument || iframe.contentWindow.document;
            } catch(e) {
                // Защита от Cross-Origin ошибок, если упали на страницу ошибки провайдера
                resolve(status); return;
            }

            // Цикл перевода по всем настроенным полям
            for (const field of FIELDS_TO_TRANSLATE) {
                const targetInput = iframeDoc.querySelector(field.selector);
                
                if (!targetInput) {
                    console.warn(`[ID ${postId}][Внимание] Поле "${field.name}" не найдено. Пропускаем.`);
                    continue;
                }

                if (!targetInput.value || targetInput.value.trim() === "") {
                    console.log(`[ID ${postId}][Пропуск] Поле "${field.name}" пустое.`);
                    status[field.name] = 'empty';
                    continue; 
                }

                let mctWrapper = targetInput.nextElementSibling;
                if (!mctWrapper || !mctWrapper.classList.contains('mct-wrapper')) {
                    mctWrapper = targetInput.parentElement.querySelector('.mct-wrapper');
                }

                if (!mctWrapper) {
                    console.error(`[ID ${postId}][Ошибка] .mct-wrapper для "${field.name}" не найден.`);
                    continue;
                }

                const globeBtn = mctWrapper.querySelector('button.mct-globe-btn');
                if (!globeBtn) {
                    console.error(`[ID ${postId}][Ошибка] Кнопка глобуса для "${field.name}" не найдена.`);
                    continue;
                }

                console.log(`[ID ${postId}][${field.name}] Клик по глобусу.`);
                globeBtn.click();
                await delay(DELAY);

                const langBtn = mctWrapper.querySelector(`.mct-dropdown button[data-code="${TARGET_LANG}"]`);
                if (!langBtn) {
                    console.error(`[ID ${postId}][Ошибка] Язык "${TARGET_LANG}" для "${field.name}" не найден.`);
                    continue;
                }

                console.log(`[ID ${postId}][${field.name}] Клик по языку.`);
                langBtn.click();
                
                const success = await waitForSuccessClass(globeBtn, postId, field.name);
                if (success) status[field.name] = 'ok';
                await delay(DELAY);
            }

            // ФИНАЛЬНОЕ СОХРАНЕНИЕ
            //кнопка обновить для товара, поста, страницы
            //const publishBtn = iframeDoc.querySelector('input#publish');

            //кнопка обновить для категории
            const publishBtn = iframeDoc.querySelector('.edit-tag-actions input[type="submit"]');



            if (!publishBtn) {
                console.error(`[ID ${postId}][Ошибка] Кнопка "Обновить" не найдена.`);
                resolve(status); return;
            }

            console.log(`[ID ${postId}][Сохранение] Клик по "Обновить".`);
            publishBtn.click();
            
            await delay(DELAY); 
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
