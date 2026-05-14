
# 🌈 Apps scripts для гугл таблиц


## 📦 Функция-переводчик контента, умеет работать c шорткодами и html. Пример: =CUSTOM_TRANSLATE(A1; 'ru'; 'uk', strong)

```text
// =============================================================================
// CUSTOM_TRANSLATE — универсальный переводчик контента для Google Sheets
// Версия 1.2
//
// Использование в ячейке:
//   =CUSTOM_TRANSLATE(A1; "uk"; "en")          — базовый вариант
//   =CUSTOM_TRANSLATE(A1; "ru"; "uk"; "і")     — с буквой-маркером
//
// Аргументы:
//   1. content  — ячейка с контентом (текст, HTML, WP шорткоды и их смеси)
//   2. fromLang — ISO-код языка источника ("uk", "ru", "en" и т.д.)
//   3. toLang   — ISO-код языка назначения
//   4. letter   — [необязательно] буква-маркер для пропуска перевода.
//                 Если текст содержит эту букву хотя бы один раз —
//                 функция считает его уже переведённым и возвращает исходник.
//                 Пример: буква "і" есть только в украинском алфавите,
//                 поэтому её наличие означает что текст уже на украинском.
//
// Поддерживаемые форматы контента:
//   - Простой текст
//   - HTML (включая битый — будет попытка восстановления)
//   - WP шорткоды в квадратных скобках (все в скобках, делятся на два вида):
//       · Одиночные — не переводим целиком:
//           [contact-form-7 id="102c92f" title="button-ques"]
//       · Парные — переводим только содержимое между ними:
//           [vc_column_text]...текст...[/vc_column_text]
//   - Смешанный контент: шорткоды + HTML внутри
//   - HTML-комментарии <!-- ... -->
//   - Одиночные (void) HTML-теги: <img />, <br />, <hr /> — не переводятся
//   - Классы с квадратными скобками (Tailwind): rounded px-px py-[0.2rem] — сохраняются
// =============================================================================


/**
 * Главная точка входа — кастомная функция для ячейки Google Sheets.
 *
 * @param {string} content         Содержимое ячейки для перевода
 * @param {string} fromLang        Язык источника (ISO 639-1, например "uk")
 * @param {string} toLang          Язык назначения (ISO 639-1, например "en")
 * @param {string} [letter]        Необязательно. Если указана буква (или строка),
 *                                 и исходная ячейка содержит её хотя бы один раз —
 *                                 перевод пропускается и возвращается исходный текст.
 *                                 Полезно для детектирования уже переведённого контента.
 *                                 Пример: =CUSTOM_TRANSLATE(A1;"ru";"uk";"і")
 *                                 → если в A1 есть "і", значит текст уже украинский — не трогаем.
 * @return {string} Переведённый контент с сохранёнными тегами
 * @customfunction
 */
function CUSTOM_TRANSLATE(content, fromLang, toLang, letter) {
  // --- Базовые проверки аргументов ---
  if (!content || String(content).trim() === "") return "";
  if (!fromLang || !toLang) return "❌ Укажи языки (аргументы 2 и 3)";
  if (fromLang === toLang) return String(content); // нет смысла переводить

  const text = String(content);

  // --- Проверка буквы-маркера (4-й необязательный аргумент) ---
  // Если буква задана и встречается в тексте — контент уже на целевом языке, пропускаем перевод.
  if (letter !== undefined && letter !== null && String(letter).length > 0) {
    if (text.includes(String(letter))) return text;
  }

  // --- Проверка доступности сервиса перевода ---
  if (!isTranslationWorking_(toLang)) {
    return "⚠️ Сервис перевода недоступен (лимит исчерпан или ошибка сети)";
  }

  try {
    // --- Основной пайплайн ---
    const healed   = healHtml_(text);          // 1. Пытаемся починить битый HTML
    const segments = parseSegments_(healed);   // 2. Разбиваем на переводимые/непереводимые куски
    const result   = translateSegments_(segments, fromLang, toLang); // 3. Переводим только текст
    return assembleResult_(result);            // 4. Собираем обратно
  } catch (e) {
    return "❌ Ошибка: " + e.message;
  }
}


// =============================================================================
// 1. ПРОВЕРКА ДОСТУПНОСТИ СЕРВИСА
// =============================================================================

/**
 * Отправляет тестовое слово в LanguageApp чтобы убедиться что сервис работает.
 * Если получаем то же слово обратно или ошибку — сервис недоступен.
 *
 * @param {string} toLang Целевой язык для проверки
 * @return {boolean} true — сервис работает, false — недоступен
 * @private
 */
function isTranslationWorking_(toLang) {
  try {
    const testWord = "Hello";
    // Переводим с английского на целевой язык
    const result = LanguageApp.translate(testWord, "en", toLang).toLowerCase().trim();
    // Если получили обратно "hello" — перевод не сработал (сервис вернул исходник)
    return result !== "hello";
  } catch (e) {
    return false;
  }
}


// =============================================================================
// 2. ЛЕЧЕНИЕ БИТОГО HTML
// =============================================================================

/**
 * Пытается исправить типичные проблемы в HTML-разметке:
 *   - Не закрытые теги (добавляем закрывающие в конец)
 *   - Пропущенные кавычки в атрибутах (упрощённо)
 *   - Двойные открытия без закрытия
 *
 * Важно: не трогаем WP шорткоды вида [тег] — они не HTML.
 * Tailwind-классы с [...] внутри атрибутов тоже не трогаем.
 *
 * @param {string} html Исходный контент
 * @return {string} Контент с исправленным HTML (или исходник если не смогли)
 * @private
 */
function healHtml_(html) {
  // Список HTML void-тегов (одиночные, не нуждаются в закрытии)
  const VOID_TAGS = new Set([
    "area","base","br","col","embed","hr","img","input",
    "link","meta","param","source","track","wbr"
  ]);

  try {
    // Стек открытых тегов для отслеживания вложенности
    const stack = [];
    // Регулярка: ищем HTML-теги, но не трогаем WP шорткоды [...]
    // Паттерн захватывает: открывающие, закрывающие и самозакрывающиеся теги
    const tagPattern = /<\/?([a-zA-Z][a-zA-Z0-9]*)\b[^>]*\/?>/g;
    let match;

    while ((match = tagPattern.exec(html)) !== null) {
      const full    = match[0];
      const tagName = match[1].toLowerCase();

      if (VOID_TAGS.has(tagName)) continue; // void-теги пропускаем

      if (full.startsWith("</")) {
        // Закрывающий тег — убираем из стека
        const idx = stack.lastIndexOf(tagName);
        if (idx !== -1) stack.splice(idx, 1);
      } else if (!full.endsWith("/>")) {
        // Открывающий тег (не самозакрывающийся)
        stack.push(tagName);
      }
    }

    // Добавляем недостающие закрывающие теги в конец (в обратном порядке)
    let healed = html;
    for (let i = stack.length - 1; i >= 0; i--) {
      healed += "</" + stack[i] + ">";
    }

    return healed;
  } catch (e) {
    // Если лечение упало — возвращаем оригинал
    return html;
  }
}


// =============================================================================
// 3. ПАРСИНГ СЕГМЕНТОВ
// =============================================================================

/**
 * Разбивает контент на массив сегментов двух типов:
 *   { type: "text", value: "..." }  — нужно переводить
 *   { type: "tag",  value: "..." }  — НЕ переводим, сохраняем как есть
 *
 * Обрабатываемые «теги» (тип "tag"):
 *   - HTML-теги: <div class="...">, </div>, <br />, <!-- комментарий -->
 *   - WP шорткоды со скобками: [vc_row], [/vc_column_text], [vc_raw_html][/vc_raw_html]
 *
 * НЕ считаем тегами:
 *   - Квадратные скобки внутри атрибутов (Tailwind): class="py-[0.2rem]"
 *     → они окажутся внутри HTML-тега и будут захвачены его паттерном целиком.
 *
 * @param {string} content Контент после healHtml_
 * @return {Array<{type:string, value:string}>} Массив сегментов
 * @private
 */
function parseSegments_(content) {
  const segments = [];

  // Общий паттерн — порядок альтернатив важен:
  //  1. HTML-комментарии  <!-- ... -->
  //  2. HTML-теги         <tag ...> или </tag> или <tag ... />
  //     Tailwind-скобки py-[0.2rem] внутри атрибутов захватываются вместе с тегом — не страшно.
  const TOKEN_RE = /<!--[\s\S]*?-->|<\/?[a-zA-Z][a-zA-Z0-9]*\b[^>]*\/?>/g;

  // WP шорткоды обрабатываются отдельным проходом по текстовым фрагментам (см. pushTextWithShortcodes_).
  // Это гарантирует что [ внутри HTML-атрибутов уже «съедены» TOKEN_RE и до WP_RE не доберутся.

  let lastIndex = 0;
  let match;

  // --- Первый проход: HTML-теги и комментарии ---
  while ((match = TOKEN_RE.exec(content)) !== null) {
    const start = match.index;
    const end   = match.index + match[0].length;

    // Текст перед тегом
    if (start > lastIndex) {
      const rawText = content.slice(lastIndex, start);
      // Разбиваем текстовый кусок на подсегменты: WP шорткоды vs обычный текст
      pushTextWithShortcodes_(rawText, segments);
    }

    // Сам тег/комментарий — не переводим
    segments.push({ type: "tag", value: match[0] });
    lastIndex = end;
  }

  // Остаток после последнего тега
  if (lastIndex < content.length) {
    const rawText = content.slice(lastIndex);
    pushTextWithShortcodes_(rawText, segments);
  }

  return segments;
}

/**
 * Вспомогательная: разбивает текстовый кусок на WP шорткоды и обычный текст.
 * Добавляет сегменты в переданный массив.
 *
 * WP шорткоды бывают двух видов:
 *   · Закрывающие:  [/vc_column_text]
 *   · Открывающие / одиночные: [contact-form-7 id="102c92f" title="button-ques"]
 *                               [vc_row]
 *
 * Паттерн разбит на две альтернативы (порядок важен):
 *   1. Закрывающий:  \[\/name\]
 *   2. Открывающий:  \[name\] или \[name атрибуты\]
 *      Атрибуты могут содержать: слова, числа, дефисы, знак =,
 *      строки в кавычках (включая спецсимволы внутри) и пробелы.
 *      Паттерн атрибутов:  (?:\s+(?:[^\[\]"']|"[^"]*"|'[^']*'))*
 *        — захватывает любые символы кроме [ ] ' ", а также строки в одинарных
 *          и двойных кавычках (id="102c92f" title="Форма зв'язку" и т.п.)
 *
 * @param {string} text     Текстовый фрагмент (без HTML-тегов)
 * @param {Array}  segments Массив сегментов (мутируем)
 * @private
 */
function pushTextWithShortcodes_(text, segments) {
  // Закрывающий шорткод: [/name]
  // Открывающий/одиночный: [name] или [name attr="val" attr2='val2']
  //   Атрибутная часть: ноль или более групп: пробел(ы) + (символ вне []'" | "строка" | 'строка')
  const WP_RE = /\[\/[a-zA-Z][a-zA-Z0-9_-]*\]|\[[a-zA-Z][a-zA-Z0-9_-]*(?:\s+(?:[^\[\]"']|"[^"]*"|'[^']*'))*\s*\]/g;
  let last = 0;
  let m;

  while ((m = WP_RE.exec(text)) !== null) {
    if (m.index > last) {
      segments.push({ type: "text", value: text.slice(last, m.index) });
    }
    segments.push({ type: "tag", value: m[0] }); // WP шорткод — не переводим
    last = m.index + m[0].length;
  }

  if (last < text.length) {
    segments.push({ type: "text", value: text.slice(last) });
  }
}


// =============================================================================
// 4. ПЕРЕВОД СЕГМЕНТОВ
// =============================================================================

/**
 * Проходит по массиву сегментов и переводит только те, у которых type === "text".
 * Пустые строки и строки только из пробелов/переносов не отправляет в API.
 *
 * @param {Array<{type:string, value:string}>} segments
 * @param {string} fromLang
 * @param {string} toLang
 * @return {Array<{type:string, value:string}>} Сегменты с переведёнными текстами
 * @private
 */
function translateSegments_(segments, fromLang, toLang) {
  return segments.map(function(seg) {
    if (seg.type !== "text") return seg; // теги не трогаем

    const trimmed = seg.value.replace(/\n/g, "").replace(/\r/g, "").trim();
    if (trimmed === "") return seg; // пустой/пробельный текст не переводим

    try {
      const translated = LanguageApp.translate(seg.value, fromLang, toLang);
      // Сохраняем лидирующие/завершающие пробелы и переносы строк оригинала
      const leading  = seg.value.match(/^[\s\n\r]*/)[0];
      const trailing = seg.value.match(/[\s\n\r]*$/)[0];
      // LanguageApp иногда обрезает пробелы — восстанавливаем
      return { type: "text", value: leading + translated.trim() + trailing };
    } catch (e) {
      // Если перевод упал для этого куска — оставляем оригинал
      return seg;
    }
  });
}


// =============================================================================
// 5. СБОРКА РЕЗУЛЬТАТА
// =============================================================================

/**
 * Склеивает все сегменты обратно в одну строку.
 *
 * @param {Array<{type:string, value:string}>} segments
 * @return {string}
 * @private
 */
function assembleResult_(segments) {
  return segments.map(function(s) { return s.value; }).join("");
}


```
