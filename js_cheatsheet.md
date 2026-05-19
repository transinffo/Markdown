# 🌈 JavaScript Cheat Sheet

## 📦 Программно кликаем по переводу заголовка и контента поста

```js
// ==================== НАСТРОЙКИ (КОНФИГ) ====================
const TARGET_LANG = 'ru'; // Код целевого языка ('uk' или 'ru')
const DELAY = 2000;       // Единая задержка для всего (клики, ожидания) в мс

// Шаблон URL. Обязательно оставляй {post_id} там, где должен быть ID страницы
const URL_TEMPLATE = 'https://test.dronestore.com.ua/wp-admin/post.php?post={post_id}&action=edit';

// Селекторы элементов, которые нужно перевести последовательно.
const FIELDS_TO_TRANSLATE = [
    { name: 'Заголовок', selector: '#title' },
    { name: 'Контент', selector: '#content' },
    { name: 'Краткое описание', selector: '#excerpt' },
    { name: 'Кастомный таб заголовок', selector: '#custom_tab1_title' },
    { name: 'Кастомный таб контент', selector: '#custom_tab1' }
];

// Твой массив ID постов
const POST_IDS = [
    25012, 25013, 25011, 25009, 25010, 25008, 25006, 25007, 25004, 25005, 25003, 25001, 25002, 25000, 24998, 24999, 24996, 24997, 24994, 24995, 24992, 24993, 24991, 24990, 24988, 24989, 24987, 24986, 24984, 24985, 24983, 24981, 24982, 24979, 24980, 24978, 24976, 24977, 24975, 24973, 24974, 24972, 24971, 24969, 24970, 24968, 24966, 24967, 24965, 24964
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
    // Фиксируем время старта скрипта
    const startTime = performance.now();

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

    // Завершение работы с DOM
    iframe.remove();
    
    // Вычисляем затраченное время
    const endTime = performance.now();
    const timeSpentMs = endTime - startTime;
    const minutes = Math.floor(timeSpentMs / 60000);
    const seconds = Math.floor((timeSpentMs % 60000) / 1000);

    console.log('%c\n[УСПЕХ] Робот закончил работу! Итоговый отчет ниже:', 'color: #46b450; font-weight: bold; font-size: 14px;');
    
    // Вывод красивой таблицы результатов
    console.table(result_array);

    // Вывод общего времени выполнения
    console.log(
        `%c⏱️ Общее время работы скрипта: ${minutes} мин. ${seconds} сек.`, 
        'color: #0073aa; font-weight: bold; font-size: 13px; background: #f0f6fb; padding: 8px; border-left: 4px solid #11a0d2;'
    );
}

// Функция обработки одного конкретного поста
async function processSinglePost(iframe, postId) {
    const status = {
        post_id: postId,
        page_download: 'failed'
    };
    
    // Предзаполняем поля статусом 'skipped'
    FIELDS_TO_TRANSLATE.forEach(field => {
        status[field.name] = 'skipped';
    });
    status['page_save'] = 'skipped';

    return new Promise((resolve) => {
        iframe.src = URL_TEMPLATE.replace('{post_id}', postId);

        iframe.onload = async () => {
            console.log(`[ID ${postId}] Страница загружена. Ожидаем инициализацию...`);
            status.page_download = 'ok';
            await delay(DELAY);

            const iframeDoc = iframe.contentDocument || iframe.contentWindow.document;

            // Цикл перевода по всем настроенным полям
            for (const field of FIELDS_TO_TRANSLATE) {
                const targetInput = iframeDoc.querySelector(field.selector);
                
                if (!targetInput) {
                    console.warn(`[ID ${postId}][Внимание] Поле "${field.name}" не найдено на странице. Пропускаем.`);
                    continue;
                }

                // Проверка на пустоту
                if (!targetInput.value || targetInput.value.trim() === "") {
                    console.log(`[ID ${postId}][Пропуск] Поле "${field.name}" пустое. Переходим дальше.`);
                    status[field.name] = 'empty';
                    continue; 
                }

                // Ищем соседний блок .mct-wrapper
                let mctWrapper = targetInput.nextElementSibling;
                if (!mctWrapper || !mctWrapper.classList.contains('mct-wrapper')) {
                    mctWrapper = targetInput.parentElement.querySelector('.mct-wrapper');
                }

                if (!mctWrapper) {
                    console.error(`[ID ${postId}][Ошибка] Блок перевода .mct-wrapper для поля "${field.name}" не найден.`);
                    continue;
                }

                // Ищем глобус
                const globeBtn = mctWrapper.querySelector('button.mct-globe-btn');
                if (!globeBtn) {
                    console.error(`[ID ${postId}][Ошибка] Кнопка глобуса для поля "${field.name}" не найдена.`);
                    continue;
                }

                console.log(`[ID ${postId}][${field.name}] Клик по глобусу.`);
                globeBtn.click();
                await delay(DELAY);

                // Ищем кнопку языка
                const langBtn = mctWrapper.querySelector(`.mct-dropdown button[data-code="${TARGET_LANG}"]`);
                if (!langBtn) {
                    console.error(`[ID ${postId}][Ошибка] Язык "${TARGET_LANG}" для поля "${field.name}" не найден.`);
                    continue;
                }

                console.log(`[ID ${postId}][${field.name}] Клик по языку.`);
                langBtn.click();
                
                // Ждем окончания перевода
                const success = await waitForSuccessClass(globeBtn, postId, field.name);
                if (success) {
                    status[field.name] = 'ok';
                }
                await delay(DELAY);
            }

            // ФИНАЛЬНОЕ СОХРАНЕНИЕ
            const publishBtn = iframeDoc.querySelector('input#publish');
            if (!publishBtn) {
                console.error(`[ID ${postId}][Ошибка] Кнопка "Обновить" не найдена. Сохранение невозможно.`);
                resolve(status); return;
            }

            console.log(`[ID ${postId}][Сохранение] Клик по "Обновить".`);
            publishBtn.click();
            
            // Двойная задержка на запись в БД
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
