# 🌈 JavaScript Cheat Sheet

## 📦 Красивый вывод корневой структуры репозитория гитхаба

```js
//Сохранить токен один раз
//localStorage.setItem('gh_token', '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');

(async (options = {}) => {
  const match = location.pathname.match(/^\/([^\/]+)\/([^\/]+)/);
  if (!match) return console.log('Not a GitHub repo page');

  const owner = match[1];
  const repo = match[2].replace(/\/.*/, '');

  // 🔑 токен: options → localStorage → null
  const token =
    options.token ||
    localStorage.getItem('gh_token') ||
    null;

  const {
    maxDepth = Infinity,
    ignore = ['node_modules', '.git', 'vendor', 'build']
  } = options;

  const base = `https://api.github.com/repos/${owner}/${repo}/contents`;

  let output = '';

  const headers = token
    ? { Authorization: `token ${token}` }
    : {};

  const walk = async (url, prefix = '', depth = 0) => {
    if (depth > maxDepth) return;

    const res = await fetch(url, { headers });
    const data = await res.json();

    if (!Array.isArray(data)) {
      console.log('API error:', data);
      return;
    }

    const filtered = data.filter(i =>
      !ignore.some(skip => i.path?.includes(skip) || i.name?.includes(skip))
    );

    for (let i = 0; i < filtered.length; i++) {
      const item = filtered[i];
      const isLast = i === filtered.length - 1;

      const branch = isLast ? '└── ' : '├── ';
      const nextPrefix = prefix + (isLast ? '    ' : '│   ');

      output += prefix + branch + item.name + '\n';

      if (item.type === 'dir') {
        await walk(item.url, nextPrefix, depth + 1);
      }
    }
  };

  try {
    await walk(base);

    console.log(output);

    if (navigator.clipboard?.writeText) {
      await navigator.clipboard.writeText(output);
      console.log('✅ Copied to clipboard');
    } else {
      console.log('📋 Copy manually');
    }

  } catch (e) {
    console.error('❌ Error:', e);
  }
})();
```

## 📦 Красивый вывод в консоль браузера массива _FAST_CACHE

```js
(() => {
    if (window.ApolloCacheState && window.ApolloCacheState._FAST_CACHE) {
        const cacheData = window.ApolloCacheState._FAST_CACHE;
        
        // Превращаем объект в красивую JSON-строку с отступом в 4 пробела
        const prettyJson = JSON.stringify(cacheData, null, 4);
        
        console.clear();
        console.log("%c--- КРАСИВЫЙ JSON (ОБЪЕКТ _FAST_CACHE) ---", "color: #00ff00; font-weight: bold;");
        console.log(prettyJson);
        
        // Автоматически копируем этот JSON в буфер обмена, чтобы вы могли сразу вставить его в текстовый редактор
        copy(prettyJson);
        console.log("%c[Успех] Весь JSON скопирован в буфер обмена! Нажмите Ctrl+V в любом редакторе.", "color: #1e90ff;");
        
    } else {
        console.error("window.ApolloCacheState._FAST_CACHE не найден.");
    }
})();
```

## 📦 Программно кликаем по переводу заголовка и контента поста версия 1.2 (логи в строку)

```js
// ==================== НАСТРОЙКИ (КОНФИГ) ====================
// tvoe.misto.2012
// 7tZY*luf1A*4AFfwB42ByFa^
const TARGET_LANG = 'ru'; // Код целевого языка ('uk' или 'ru')
const DELAY = 1000;       // Единая задержка для всего (клики, ожидания) в мс
const MAX_ATTEMPTS = 3;   // Количество попыток загрузки страницы при сбое сети
const RETRY_DELAY = 15000; // Сколько ждать (в мс) перед повторной попыткой при ошибке

// --- ВАРИАНТ 1: Для товаров, постов и страниц ---
const URL_TEMPLATE = 'https://test.dronestore.com.ua/wp-admin/post.php?post={post_id}&action=edit';
const FIELDS_TO_TRANSLATE = [
     { name: 'Заголовок', key: 'title', selector: '#title' },
     { name: 'Контент', key: 'content', selector: '#content' },
     { name: 'Краткое описание', key: 'excerpt', selector: '#excerpt' },
     { name: 'Кастомный таб заголовок', key: 'custom_tab1_title', selector: '#custom_tab1_title' },
     { name: 'Кастомный таб контент', key: 'custom_tab1', selector: '#custom_tab1' }
];
const PUBLISH_BTN_SELECTOR = 'input#publish'; // Селектор кнопки для товаров/постов

// --- ВАРИАНТ 2: Для категорий WooCommerce ---
// const URL_TEMPLATE = 'https://test.dronestore.com.ua/wp-admin/term.php?taxonomy=product_cat&tag_ID={post_id}&post_type=product';
// const FIELDS_TO_TRANSLATE = [
//      { name: 'Заголовок', key: 'title', selector: '#name' },
//      { name: 'Описание', key: 'content', selector: '#description' }
// ];
// const PUBLISH_BTN_SELECTOR = '.edit-tag-actions input[type="submit"]'; // Селектор кнопки для категорий


// Твой массив ID постов
const POST_IDS = [
23460, 23457, 23458, 23456, 23455, 23453, 23454, 23451, 23452, 23450, 23449, 23447, 23448, 23446, 23445, 23443, 23444, 23442, 23441, 23440, 23439, 23437, 23438, 23436, 23435, 23433, 23434, 23431, 23432, 23429, 23430, 23427, 23428, 23426, 23425, 23423, 23424, 23422, 23421, 23420, 23419, 23417, 23418, 23416, 23414, 23415, 23413, 23411, 23412, 23410, 23409, 23407, 23408, 23406, 23404, 23405, 23402, 23403, 23401, 23399, 23400, 23398, 23396, 23397, 23395, 23393, 23394, 23391, 23392, 23390, 23388, 23389, 23386, 23387, 23385, 23383, 23384, 23381, 23382, 23379, 23380, 23378, 23376, 23377, 23375, 23374, 23372, 23373, 23371, 23369, 23370, 23368, 23366, 23367, 23365, 23363, 23364, 23361, 23362, 23359, 23360, 23357, 23358, 23356, 23354, 23355, 23353, 23351, 23352, 23350, 23348, 23349, 23346, 23347, 23344, 23345, 23343, 23341, 23342, 23339, 23340, 23337, 23338, 23336, 23334, 23335, 23333, 23331, 23332, 23329, 23330, 23327, 23328, 23326, 23324, 23325, 23322, 23323, 23321, 23320, 23319, 23317, 23318, 23316, 23313, 23314, 23315, 23311, 23312, 23309, 23310, 23308, 23306, 23307, 23304, 23305, 23302, 23303, 23301, 23299, 23300, 23297, 23298, 23296, 23295, 23293, 23294, 23291, 23292, 23289, 23290, 23288, 23286, 23287, 23284, 23285, 23283, 23281, 23282, 23279, 23280, 23278, 23276, 23277, 23274, 23275, 23272, 23273, 23271, 23270, 23268, 23269, 23267, 23266, 23265, 23262, 23263, 23264, 23261, 23259, 23260, 23256, 23257, 23258, 23254, 23255, 23252, 23253, 23251, 23250, 23249, 23247, 23248, 23246, 23245, 23244, 23242, 23243, 23241, 23240, 23239, 23236, 23237, 23238
];
// ============================================================

const STORAGE_KEY_RESULTS = 'wp_automation_results';
const STORAGE_KEY_DONE_IDS = 'wp_automation_done_ids';

const result_array = JSON.parse(localStorage.getItem(STORAGE_KEY_RESULTS)) || [];
const done_post_ids = new Set(JSON.parse(localStorage.getItem(STORAGE_KEY_DONE_IDS)) || []);

const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));

async function waitForTranslatedClass(element, timeout = 10000) {
    return new Promise((resolve) => {
        if (element.classList.contains('translated')) {
            resolve(true);
            return;
        }

        const timer = setTimeout(() => {
            observer.disconnect();
            resolve(false);
        }, timeout);

        const observer = new MutationObserver((mutationsList) => {
            for (const mutation of mutationsList) {
                if (mutation.type === 'attributes' && mutation.attributeName === 'class') {
                    if (element.classList.contains('translated')) {
                        clearTimeout(timer);
                        observer.disconnect();
                        resolve(true);
                    }
                }
            }
        });

        observer.observe(element, { attributes: true, attributeFilter: ['class'] });
    });
}

async function runMassAutomation() {
    const startTime = performance.now();
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

    for (let i = 0; i < remainingIds.length; i++) {
        const postId = remainingIds[i];
        let logEntry = null;
        let attempt = 1;
        let success = false;

        while (attempt <= MAX_ATTEMPTS && !success) {
            try {
                if (attempt > 1) {
                    await delay(RETRY_DELAY);
                }
                
                logEntry = await processSinglePost(iframe, postId);
                
                if (logEntry.page_download.startsWith('error')) {
                    throw new Error(logEntry.page_download);
                }
                
                success = true;
            } catch (error) {
                console.error(`[ID ${postId}][Попытка ${attempt}] Сбой: ${error.message}`);
                attempt++;
            }
        }

        if (!success) {
            logEntry = { post_id: postId, page_download: 'error (timeout/network)' };
            FIELDS_TO_TRANSLATE.forEach(f => logEntry[f.key || f.name] = 'error (page not loaded)');
            logEntry.page_save = 'error (not saved)';
        }

        // Строим красивую единую строку лога согласно ТЗ
        const logStringParts = [`${postId} Load - ${logEntry.page_download}`];
        FIELDS_TO_TRANSLATE.forEach(f => {
            const key = f.key || f.name;
            logStringParts.push(`${key} - ${logEntry[key]}`);
        });
        logStringParts.push(`save - ${logEntry.page_save}`);
        
        // Выводим скомпилированную строку в консоль
        console.log(`%c${logStringParts.join(', ')}`, success ? 'color: #46b450;' : 'color: #dc3232; font-weight: bold;');

        result_array.push(logEntry);
        done_post_ids.add(postId);
        
        localStorage.setItem(STORAGE_KEY_RESULTS, JSON.stringify(result_array));
        localStorage.setItem(STORAGE_KEY_DONE_IDS, JSON.stringify(Array.from(done_post_ids)));
    }

    iframe.remove();
    
    const totalEndTime = performance.now();
    const timeSpentMs = totalEndTime - startTime;
    const minutes = Math.floor(timeSpentMs / 60000);
    const seconds = Math.floor((timeSpentMs % 60000) / 1000);

    console.log('%c\n[УСПЕХ] Робот закончил работу! Итоговый отчет ниже:', 'color: #46b450; font-weight: bold; font-size: 14px;');
    console.table(result_array);
    console.log(`%c⏱️ Время текущей сессии: ${minutes} мин. ${seconds} сек.`, 'color: #0073aa; font-weight: bold; font-size: 13px; background: #f0f6fb; padding: 8px; border-left: 4px solid #11a0d2;');

    clearProgress();
}

function clearProgress() {
    localStorage.removeItem(STORAGE_KEY_RESULTS);
    localStorage.removeItem(STORAGE_KEY_DONE_IDS);
}

async function processSinglePost(iframe, postId) {
    const status = { post_id: postId, page_download: 'error (init)' };
    FIELDS_TO_TRANSLATE.forEach(field => status[field.key || field.name] = 'error (init)');
    status['page_save'] = 'error (init)';

    return new Promise((resolve) => {
        const networkTimeout = setTimeout(() => {
            status.page_download = 'error (timeout)';
            resolve(status); 
        }, 30000);

        iframe.src = URL_TEMPLATE.replace('{post_id}', postId);

        iframe.onload = async () => {
            clearTimeout(networkTimeout);
            status.page_download = 'ok';
            await delay(DELAY);

            let iframeDoc;
            try {
                iframeDoc = iframe.contentDocument || iframe.contentWindow.document;
                if (!iframeDoc) throw new Error('CORS or blank document');
            } catch(e) {
                status.page_download = 'error (iframe blocked)';
                resolve(status); 
                return;
            }

            let needSave = false;

            for (const field of FIELDS_TO_TRANSLATE) {
                const key = field.key || field.name;
                const targetInput = iframeDoc.querySelector(field.selector);
                
                if (!targetInput) {
                    status[key] = 'error (not found)';
                    continue;
                }

                const value = targetInput.value ? targetInput.value.trim() : "";

                if (value === "") {
                    status[key] = 'latin'; // Пустые поля логически эквивалентны латинице (пропускаем)
                    continue; 
                }

                const hasCyrillic = /[а-яёієїґ]/i.test(value);
                if (!hasCyrillic) {
                    status[key] = 'latin';
                    continue; 
                }

                needSave = true;

                let mctWrapper = targetInput.nextElementSibling;
                if (!mctWrapper || !mctWrapper.classList.contains('mct-wrapper')) {
                    mctWrapper = targetInput.parentElement.querySelector('.mct-wrapper');
                }

                if (!mctWrapper) {
                    status[key] = 'error (wrapper missing)';
                    continue;
                }

                const globeBtn = mctWrapper.querySelector('button.mct-globe-btn');
                if (!globeBtn) {
                    status[key] = 'error (globe missing)';
                    continue;
                }

                globeBtn.click();
                await delay(DELAY);

                const langBtn = mctWrapper.querySelector(`.mct-dropdown button[data-code="${TARGET_LANG}"]`);
                if (!langBtn) {
                    status[key] = `error (lang ${TARGET_LANG} missing)`;
                    continue;
                }

                langBtn.click();
                
                const isTranslated = await waitForTranslatedClass(targetInput, 15000);
                if (isTranslated) {
                    status[key] = 'ok';
                } else {
                    status[key] = 'error (timeout waiting translate)';
                }
                await delay(DELAY);
            }

            if (!needSave) {
                status.page_save = 'latin';
                resolve(status);
                return;
            }

            const publishBtn = iframeDoc.querySelector(PUBLISH_BTN_SELECTOR);
            if (!publishBtn) {
                status.page_save = 'error (btn missing)';
                resolve(status); 
                return;
            }

            try {
                publishBtn.click();
                await delay(5000); 
                status.page_save = 'ok';
            } catch (e) {
                status.page_save = `error (${e.message.slice(0, 20)})`;
            }

            resolve(status);
        };
    });
}

// Запуск автоматизации
runMassAutomation();
```

## 📦 Программно кликаем по переводу заголовка и контента поста версия 1.1

```js
// ==================== НАСТРОЙКИ (КОНФИГ) ====================
// tvoe.misto.2012
// 7tZY*luf1A*4AFfwB42ByFa^
const TARGET_LANG = 'ru'; // Код целевого языка ('uk' или 'ru')
const DELAY = 1000;       // Единая задержка для всего (клики, ожидания) в мс
const MAX_ATTEMPTS = 3;   // Количество попыток загрузки страницы при сбое сети
const RETRY_DELAY = 15000; // Сколько ждать (в мс) перед повторной попыткой при ошибке

// --- ВАРИАНТ 1: Для товаров, постов и страниц ---
const URL_TEMPLATE = 'https://test.dronestore.com.ua/wp-admin/post.php?post={post_id}&action=edit';
const FIELDS_TO_TRANSLATE = [
     { name: 'Заголовок', selector: '#title' },
     { name: 'Контент', selector: '#content' },
     { name: 'Краткое описание', selector: '#excerpt' },
     { name: 'Кастомный таб заголовок', selector: '#custom_tab1_title' },
     { name: 'Кастомный таб контент', selector: '#custom_tab1' }
];
const PUBLISH_BTN_SELECTOR = 'input#publish'; // Селектор кнопки для товаров/постов

// --- ВАРИАНТ 2: Для категорий WooCommerce ---
// const URL_TEMPLATE = 'https://test.dronestore.com.ua/wp-admin/term.php?taxonomy=product_cat&tag_ID={post_id}&post_type=product';
// const FIELDS_TO_TRANSLATE = [
//     { name: 'Заголовок', selector: '#name' },
//     { name: 'Описание', selector: '#description' }
// ];
// const PUBLISH_BTN_SELECTOR = '.edit-tag-actions input[type="submit"]'; // Селектор кнопки для категорий


// Твой массив ID постов
const POST_IDS = [
23460, 23457, 23458, 23456, 23455, 23453, 23454, 23451, 23452, 23450, 23449, 23447, 23448, 23446, 23445, 23443, 23444, 23442, 23441, 23440, 23439, 23437, 23438, 23436, 23435, 23433, 23434, 23431, 23432, 23429, 23430, 23427, 23428, 23426, 23425, 23423, 23424, 23422, 23421, 23420, 23419, 23417, 23418, 23416, 23414, 23415, 23413, 23411, 23412, 23410, 23409, 23407, 23408, 23406, 23404, 23405, 23402, 23403, 23401, 23399, 23400, 23398, 23396, 23397, 23395, 23393, 23394, 23391, 23392, 23390, 23388, 23389, 23386, 23387, 23385, 23383, 23384, 23381, 23382, 23379, 23380, 23378, 23376, 23377, 23375, 23374, 23372, 23373, 23371, 23369, 23370, 23368, 23366, 23367, 23365, 23363, 23364, 23361, 23362, 23359, 23360, 23357, 23358, 23356, 23354, 23355, 23353, 23351, 23352, 23350, 23348, 23349, 23346, 23347, 23344, 23345, 23343, 23341, 23342, 23339, 23340, 23337, 23338, 23336, 23334, 23335, 23333, 23331, 23332, 23329, 23330, 23327, 23328, 23326, 23324, 23325, 23322, 23323, 23321, 23320, 23319, 23317, 23318, 23316, 23313, 23314, 23315, 23311, 23312, 23309, 23310, 23308, 23306, 23307, 23304, 23305, 23302, 23303, 23301, 23299, 23300, 23297, 23298, 23296, 23295, 23293, 23294, 23291, 23292, 23289, 23290, 23288, 23286, 23287, 23284, 23285, 23283, 23281, 23282, 23279, 23280, 23278, 23276, 23277, 23274, 23275, 23272, 23273, 23271, 23270, 23268, 23269, 23267, 23266, 23265, 23262, 23263, 23264, 23261, 23259, 23260, 23256, 23257, 23258, 23254, 23255, 23252, 23253, 23251, 23250, 23249, 23247, 23248, 23246, 23245, 23244, 23242, 23243, 23241, 23240, 23239, 23236, 23237, 23238
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

// Функция-помощник: ждет появления класса "translated" у самого инпута/textarea
async function waitForTranslatedClass(element, postId, blockName) {
    return new Promise((resolve) => {
        // Если класс уже есть на элементе
        if (element.classList.contains('translated')) {
            resolve(true);
            return;
        }

        const observer = new MutationObserver((mutationsList) => {
            for (const mutation of mutationsList) {
                if (mutation.type === 'attributes' && mutation.attributeName === 'class') {
                    if (element.classList.contains('translated')) {
                        observer.disconnect();
                        console.log(`[ID ${postId}][Сделано] Поле "${blockName}" получило класс .translated.`);
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
    
    const totalEndTime = performance.now();
    const timeSpentMs = totalEndTime - startTime;
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
        const networkTimeout = setTimeout(() => {
            resolve(status); 
        }, 30000); // 30 секунд на загрузку страницы

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
                resolve(status); return;
            }

            // Флаг, который определит, нужно ли в конце нажимать кнопку "Обновить"
            let needSave = false;

            // Цикл перевода по всем настроенным полям
            for (const field of FIELDS_TO_TRANSLATE) {
                const targetInput = iframeDoc.querySelector(field.selector);
                
                if (!targetInput) {
                    console.warn(`[ID ${postId}][Внимание] Поле "${field.name}" не найдено. Пропускаем.`);
                    continue;
                }

                const value = targetInput.value ? targetInput.value.trim() : "";

                if (value === "") {
                    console.log(`[ID ${postId}][Пропуск] Поле "${field.name}" пустое.`);
                    status[field.name] = 'empty';
                    continue; 
                }

                // Проверка на латиницу: ищем, есть ли в тексте кириллица (включая укр. буквы)
                const hasCyrillic = /[а-яёієїґ]/i.test(value);
                
                if (!hasCyrillic) {
                    console.log(`[ID ${postId}][Пропуск] Поле "${field.name}" содержит только латиницу/цифры ("${value}"). Перевод не требуется.`);
                    status[field.name] = 'skipped_latin';
                    continue; // Сразу переходим к следующему полю БЕЗ задержек и кликов
                }

                // Если дошли сюда — поле требует перевода, значит страницу нужно будет сохранить
                needSave = true;

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
                
                // Ждем появления класса .translated непосредственно на элементе ввода (input/textarea)
                const success = await waitForTranslatedClass(targetInput, postId, field.name);
                if (success) status[field.name] = 'ok';
                await delay(DELAY);
            }

            // ПРОВЕРКА НА КЛИК КНОПКИ «ОБНОВИТЬ»
            if (!needSave) {
                console.log(`%c[ID ${postId}][ПРОПУСК ОБНОВЛЕНИЯ] Все поля пустые или содержат только латиницу. Переходим к следующей странице без сохранения.`, 'color: #00aa73; font-weight: bold;');
                status.page_save = 'skipped_no_changes';
                resolve(status);
                return;
            }

            // ФИНАЛЬНОЕ СОХРАНЕНИЕ СТРАНИЦЫ (выполняется, только если needSave === true)
            const publishBtn = iframeDoc.querySelector(PUBLISH_BTN_SELECTOR);

            if (!publishBtn) {
                console.error(`[ID ${postId}][Ошибка] Кнопка "Обновить" с селектором "${PUBLISH_BTN_SELECTOR}" не найдена.`);
                resolve(status); return;
            }

            console.log(`[ID ${postId}][Сохранение] Клик по "Обновить".`);
            publishBtn.click();
            
            // Даем WordPress 5 секунд на обработку сохранения базой данных перед открытием следующего ID
            await delay(5000); 
            status.page_save = 'ok';
            console.log(`[ID ${postId}][ОК] Изменения отправлены на сервер.`);

            resolve(status);
        };
    });
}

// Запуск автоматизации
runMassAutomation();
```

## 📦 Программно кликаем по переводу заголовка и контента поста версия 1.0

```js
// ==================== НАСТРОЙКИ (КОНФИГ) ====================
const TARGET_LANG = 'uk'; // Код целевого языка ('uk' или 'ru')
const DELAY = 2000;       // Единая задержка для всего (клики, ожидания) в мс
const MAX_ATTEMPTS = 3;   // Количество попыток загрузки страницы при сбое сети
const RETRY_DELAY = 15000; // Сколько ждать (в мс) перед повторной попыткой при ошибке

// Шаблон URL. Обязательно оставляй {post_id} там, где должен быть ID страницы

// ходим по страницам woo товара, постам и страницам
const URL_TEMPLATE = 'https://test.dronestore.com.ua/wp-admin/post.php?post={post_id}&action=edit';
const FIELDS_TO_TRANSLATE = [
    { name: 'Заголовок', selector: '#title' },
    { name: 'Контент', selector: '#content' },
    { name: 'Краткое описание', selector: '#excerpt' },
    { name: 'Кастомный таб заголовок', selector: '#custom_tab1_title' },
    { name: 'Кастомный таб контент', selector: '#custom_tab1' }
];

// ходим по категориям woo товара
// const URL_TEMPLATE = 'https://test.dronestore.com.ua/wp-admin/term.php?taxonomy=product_cat&tag_ID={post_id}&post_type=product';
// const FIELDS_TO_TRANSLATE = [
//     { name: 'Заголовок', selector: '#name' },
//     { name: 'Описание', selector: '#description' }
// ];



// Твой массив ID постов
const POST_IDS = [
    92, 142, 571, 1354, 1859, 1891, 1893, 2187, 3229, 5209, 5220, 5643, 6437, 6506, 7006, 7048, 7324, 7763, 8211, 8221, 9114, 9217, 9284, 9463, 9591, 9628, 9689, 10091, 17229, 17884, 19062, 19339, 20792
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
            const publishBtn = iframeDoc.querySelector('input#publish');

            //кнопка обновить для категории
            //const publishBtn = iframeDoc.querySelector('.edit-tag-actions input[type="submit"]');



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
