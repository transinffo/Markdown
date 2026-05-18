
# 🌈 Apps scripts для гугл таблиц


## 📦 Функция-переводчик контента, умеет работать c шорткодами и html. Пример: =CUSTOM_TRANSLATE(A1; 'ru'; 'uk', strong)

```js
/**
 * ============================================================
 * CUSTOM_TRANSLATE — Production-grade cell translation
 * для Google Sheets / Apps Script
 *
 * Поддерживает:
 *   - Обычный HTML (теги, атрибуты, спецсимволы, таблицы, списки)
 *   - WPBakery shortcodes (включая атрибуты типа title="...")
 *   - WordPress shortcodes (не переводятся)
 *   - HTML-комментарии (не переводятся)
 *   - Имена файлов с кириллицей в src/href (не переводятся)
 *   - Tailwind-классы внутри атрибутов class (не переводятся)
 *   - Разбивка на чанки при больших объёмах
 *   - Проверка работоспособности API перед переводом
 * ============================================================
 *
 * Использование:
 *   =CUSTOM_TRANSLATE(A1; "ru"; "uk")
 *   =CUSTOM_TRANSLATE(A1; "ru"; "uk"; "і")
 *
 * @param {string} content  — Исходный контент ячейки
 * @param {string} fromLang — Язык источника (ISO 639-1)
 * @param {string} toLang   — Язык назначения (ISO 639-1)
 * @param {string} [letter] — Буква-маркер (если найдена в тексте — возвращаем оригинал)
 * @return {string} Переведённый контент
 * @customfunction
 */
function CUSTOM_TRANSLATE(content, fromLang, toLang, letter) {

  // ── 0. Валидация входных данных ──────────────────────────────────────────
  if (!content || typeof content !== "string" || content.trim() === "") {
    return content;
  }
  if (!fromLang || !toLang) {
    throw new Error("CUSTOM_TRANSLATE: fromLang и toLang обязательны");
  }

  // ── 1. Проверка буквы-маркера ────────────────────────────────────────────
  if (letter && typeof letter === "string" && content.includes(letter)) {
    return content; // текст уже на целевом языке
  }

  // ── 2. Проверка работоспособности Translation API ────────────────────────
  if (!isTranslationWorking_(toLang)) {
    throw new Error(
      "CUSTOM_TRANSLATE: Google Translate API недоступен или достигнут дневной лимит."
    );
  }

  // ── 3. Парсим, переводим, собираем ──────────────────────────────────────
  try {
    return translateContent_(content, fromLang, toLang);
  } catch (e) {
    Logger.log("CUSTOM_TRANSLATE error: " + e.message);
    throw new Error("CUSTOM_TRANSLATE: " + e.message);
  }
}


// ═══════════════════════════════════════════════════════════════════════════
//  ВНУТРЕННИЕ ФУНКЦИИ
// ═══════════════════════════════════════════════════════════════════════════

/**
 * Проверяет работоспособность LanguageApp.
 * @param {string} toLang
 * @return {boolean}
 * @private
 */
function isTranslationWorking_(toLang) {
  try {
    const testWord = "Hello";
    const result = LanguageApp.translate(testWord, "en", toLang).toLowerCase().trim();
    return result !== "hello";
  } catch (e) {
    return false;
  }
}

// ─── Константы ─────────────────────────────────────────────────────────────

// Максимальный размер одного запроса к API (в символах)
const MAX_CHUNK_SIZE_ = 4500;

// Заглушка-разделитель для защиты токенов при переводе.
// Используем формат, который Google Translate сохраняет нетронутым.
const TOKEN_PLACEHOLDER_PREFIX_ = "XXTKNXX";
const TOKEN_PLACEHOLDER_SUFFIX_ = "XX";


/**
 * Главная функция: разбирает контент на токены, переводит текстовые,
 * собирает результат.
 * @param {string} content
 * @param {string} fromLang
 * @param {string} toLang
 * @return {string}
 * @private
 */
function translateContent_(content, fromLang, toLang) {

  // ── Шаг 1: токенизация ──────────────────────────────────────────────────
  const tokens = tokenize_(content);

  // ── Шаг 2: собираем только текстовые токены (которые нужно переводить) ──
  // Работаем батчами, чтобы не превысить лимит одного запроса
  const textTokenIndices = [];
  for (let i = 0; i < tokens.length; i++) {
    if (tokens[i].type === "TEXT" && tokens[i].value.trim() !== "") {
      textTokenIndices.push(i);
    }
  }

  if (textTokenIndices.length === 0) return content; // нечего переводить

  // ── Шаг 3: батчевый перевод ─────────────────────────────────────────────
  const translatedTexts = batchTranslate_(
    textTokenIndices.map(i => tokens[i].value),
    fromLang,
    toLang
  );

  // ── Шаг 4: подставляем переводы обратно ────────────────────────────────
  for (let j = 0; j < textTokenIndices.length; j++) {
    tokens[textTokenIndices[j]].value = translatedTexts[j];
  }

  // ── Шаг 5: собираем результат ───────────────────────────────────────────
  return tokens.map(t => t.value).join("");
}


/**
 * Батчевый перевод массива строк.
 * Разбивает на чанки по MAX_CHUNK_SIZE_ символов, каждый чанк — один запрос.
 * @param {string[]} texts
 * @param {string} fromLang
 * @param {string} toLang
 * @return {string[]}
 * @private
 */
function batchTranslate_(texts, fromLang, toLang) {
  const result = new Array(texts.length);
  const DELIMITER = "\n⏎\n"; // уникальный разделитель

  let chunkTexts = [];
  let chunkIndices = [];
  let chunkSize = 0;

  function flushChunk_() {
    if (chunkTexts.length === 0) return;

    const joined = chunkTexts.join(DELIMITER);
    let translated;
    try {
      translated = LanguageApp.translate(joined, fromLang, toLang);
    } catch (e) {
      // При ошибке возвращаем оригиналы
      Logger.log("batchTranslate_ error: " + e.message);
      translated = joined;
    }

    // Разбиваем результат по разделителю
    const parts = translated.split(DELIMITER);

    // Защита: если количество частей не совпадает — возвращаем оригиналы
    if (parts.length !== chunkTexts.length) {
      // Fallback: переводим по одному
      for (let k = 0; k < chunkIndices.length; k++) {
        try {
          result[chunkIndices[k]] = LanguageApp.translate(chunkTexts[k], fromLang, toLang);
        } catch (e2) {
          result[chunkIndices[k]] = chunkTexts[k];
        }
      }
    } else {
      for (let k = 0; k < chunkIndices.length; k++) {
        result[chunkIndices[k]] = parts[k];
      }
    }

    chunkTexts = [];
    chunkIndices = [];
    chunkSize = 0;
  }

  for (let i = 0; i < texts.length; i++) {
    const t = texts[i];
    // Если одна строка больше лимита — переводим её отдельно
    if (t.length > MAX_CHUNK_SIZE_) {
      flushChunk_();
      try {
        result[i] = LanguageApp.translate(t, fromLang, toLang);
      } catch (e) {
        result[i] = t;
      }
      continue;
    }

    const addedSize = chunkSize === 0 ? t.length : chunkSize + DELIMITER.length + t.length;
    if (addedSize > MAX_CHUNK_SIZE_ && chunkTexts.length > 0) {
      flushChunk_();
    }

    chunkTexts.push(t);
    chunkIndices.push(i);
    chunkSize = chunkSize === 0 ? t.length : chunkSize + DELIMITER.length + t.length;
  }

  flushChunk_();
  return result;
}


// ═══════════════════════════════════════════════════════════════════════════
//  ТОКЕНИЗАТОР
// ═══════════════════════════════════════════════════════════════════════════
//
// Токен = { type, value }
// type:
//   "TEXT"           — обычный текст, переводим
//   "HTML_TAG"       — HTML-тег (открывающий/закрывающий/самозакрывающийся)
//   "WPBAKERY_TAG"   — WPBakery shortcode ([ ... ]) — атрибут title переводим отдельно
//   "WP_SHORTCODE"   — обычный WP shortcode (не WPBakery, не переводим)
//   "HTML_COMMENT"   — <!-- ... --> не переводим
//   "HTML_ENTITY"    — &nbsp; &#x2705; и т.д. не переводим
//   "CDATA"          — не переводим (встречается редко)
//
// ───────────────────────────────────────────────────────────────────────────

/**
 * Разбивает исходный контент на токены.
 * @param {string} content
 * @return {{type: string, value: string}[]}
 * @private
 */
function tokenize_(content) {
  const tokens = [];
  let pos = 0;
  const len = content.length;

  while (pos < len) {

    // ── HTML-комментарий ────────────────────────────────────────────────
    if (content.startsWith("<!--", pos)) {
      const end = content.indexOf("-->", pos + 4);
      if (end !== -1) {
        tokens.push({ type: "HTML_COMMENT", value: content.slice(pos, end + 3) });
        pos = end + 3;
        continue;
      }
    }

    // ── HTML CDATA ───────────────────────────────────────────────────────
    if (content.startsWith("<![CDATA[", pos)) {
      const end = content.indexOf("]]>", pos + 9);
      if (end !== -1) {
        tokens.push({ type: "CDATA", value: content.slice(pos, end + 3) });
        pos = end + 3;
        continue;
      }
    }

    // ── HTML-тег ────────────────────────────────────────────────────────
    if (content[pos] === "<" && pos + 1 < len && content[pos + 1] !== " ") {
      const tagEnd = findHtmlTagEnd_(content, pos);
      if (tagEnd !== -1) {
        const rawTag = content.slice(pos, tagEnd);
        // Проверяем, не является ли это WP/WPBakery shortcode внутри тега (edge case)
        tokens.push({ type: "HTML_TAG", value: rawTag });
        pos = tagEnd;
        continue;
      }
    }

    // ── WPBakery / обычный WP Shortcode ─────────────────────────────────
    if (content[pos] === "[") {
      const scResult = readShortcode_(content, pos);
      if (scResult) {
        tokens.push(scResult.token);
        pos = scResult.end;
        continue;
      }
    }

    // ── HTML-сущность (&nbsp; &#x2705; и т.д.) ──────────────────────────
    if (content[pos] === "&") {
      const entityEnd = readHtmlEntity_(content, pos);
      if (entityEnd !== -1) {
        tokens.push({ type: "HTML_ENTITY", value: content.slice(pos, entityEnd) });
        pos = entityEnd;
        continue;
      }
    }

    // ── Текст: читаем до следующего специального символа ────────────────
    const textEnd = findNextSpecial_(content, pos);
    const textValue = content.slice(pos, textEnd);
    if (textValue.length > 0) {
      tokens.push({ type: "TEXT", value: textValue });
    }
    pos = textEnd;
  }

  return tokens;
}


/**
 * Находит конец HTML-тега, начинающегося с pos.
 * Корректно обрабатывает атрибуты с кавычками и Tailwind-скобки.
 * @param {string} s
 * @param {number} start — позиция '<'
 * @return {number} позиция после '>', или -1 если не нашли
 * @private
 */
function findHtmlTagEnd_(s, start) {
  const len = s.length;
  let i = start + 1;

  // Должна идти буква, '/', '!'
  if (i >= len) return -1;
  const c = s[i];
  if (!/[a-zA-Z\/!]/.test(c)) return -1;

  let inSingleQuote = false;
  let inDoubleQuote = false;
  let bracketDepth = 0; // для Tailwind [ ... ]

  while (i < len) {
    const ch = s[i];

    if (!inSingleQuote && !inDoubleQuote) {
      if (ch === '"') { inDoubleQuote = true; i++; continue; }
      if (ch === "'") { inSingleQuote = true; i++; continue; }
      if (ch === "[") { bracketDepth++; i++; continue; }
      if (ch === "]") { bracketDepth--; i++; continue; }
      if (ch === ">" && bracketDepth === 0) return i + 1;
    } else if (inDoubleQuote) {
      if (ch === '"') inDoubleQuote = false;
    } else if (inSingleQuote) {
      if (ch === "'") inSingleQuote = false;
    }
    i++;
  }
  return -1;
}


/**
 * Читает shortcode начиная с '['.
 * Определяет тип: WPBakery (vc_*) или обычный WP.
 * Для WPBakery возвращает токен со специальным типом.
 * @param {string} s
 * @param {number} start
 * @return {{token: {type:string, value:string}, end:number}|null}
 * @private
 */
function readShortcode_(s, start) {
  const len = s.length;
  let i = start + 1;

  // Закрывающий shortcode [/tag] — не переводим
  const isClosing = (s[i] === "/");
  if (isClosing) i++;

  // Читаем имя тега
  let tagName = "";
  while (i < len && /[a-zA-Z0-9_\-]/.test(s[i])) {
    tagName += s[i++];
  }

  if (!tagName) return null;

  // Найдём закрывающую ] с учётом вложенных {} (например css=".vc_custom_{...}")
  const end = findShortcodeEnd_(s, start);
  if (end === -1) return null;

  const raw = s.slice(start, end);

  // WPBakery shortcodes начинаются с vc_
  const isWPBakery = tagName.startsWith("vc_");

  return {
    token: { type: isWPBakery ? "WPBAKERY_TAG" : "WP_SHORTCODE", value: raw },
    end: end
  };
}


/**
 * Находит конец shortcode [ ... ], учитывая вложенные кавычки и фигурные скобки.
 * @param {string} s
 * @param {number} start — позиция '['
 * @return {number} позиция после ']', или -1
 * @private
 */
function findShortcodeEnd_(s, start) {
  const len = s.length;
  let i = start + 1;
  let inDoubleQuote = false;
  let inSingleQuote = false;
  let braceDepth = 0;

  while (i < len) {
    const ch = s[i];

    if (!inDoubleQuote && !inSingleQuote) {
      if (ch === '"') { inDoubleQuote = true; i++; continue; }
      if (ch === "'") { inSingleQuote = true; i++; continue; }
      if (ch === "{") { braceDepth++; i++; continue; }
      if (ch === "}") { braceDepth--; i++; continue; }
      if (ch === "]" && braceDepth === 0) return i + 1;
    } else if (inDoubleQuote) {
      if (ch === '"') inDoubleQuote = false;
    } else if (inSingleQuote) {
      if (ch === "'") inSingleQuote = false;
    }
    i++;
  }
  return -1;
}


/**
 * Читает HTML-сущность начиная с '&'.
 * @param {string} s
 * @param {number} start
 * @return {number} позиция после ';', или -1
 * @private
 */
function readHtmlEntity_(s, start) {
  // &[#][a-zA-Z0-9x]+;
  const match = s.slice(start).match(/^&(?:#x?[0-9a-fA-F]+|[a-zA-Z][a-zA-Z0-9]*);/);
  if (match) return start + match[0].length;
  return -1;
}


/**
 * Находит позицию следующего специального символа: '<', '[', '&'.
 * @param {string} s
 * @param {number} start
 * @return {number}
 * @private
 */
function findNextSpecial_(s, start) {
  const len = s.length;
  for (let i = start; i < len; i++) {
    const ch = s[i];
    if (ch === "<" || ch === "[" || ch === "&") return i;
  }
  return len;
}


// ═══════════════════════════════════════════════════════════════════════════
//  ПОСТОБРАБОТКА ТОКЕНОВ: перевод атрибутов WPBakery
// ═══════════════════════════════════════════════════════════════════════════
//
// WPBakery shortcodes хранятся как "WPBAKERY_TAG" токены.
// После основного перевода текстовых токенов нам нужно отдельно перевести
// переводимые атрибуты внутри WPBakery-тегов (title, label и подобные).
//
// Чтобы не усложнять токенизатор, реализуем это через translateContent_:
// перед сборкой финального результата проходим по WPBAKERY_TAG токенам
// и переводим их атрибуты.

/**
 * Атрибуты WPBakery, значения которых нужно переводить.
 * @type {string[]}
 */
const WPBAKERY_TRANSLATABLE_ATTRS_ = ["title", "label", "tab_title", "section_title", "description"];

/**
 * Переводит значения нужных атрибутов в WPBakery shortcode.
 * @param {string} shortcode — строка вида [vc_tta_section title="Текст" tab_id="..."]
 * @param {string} fromLang
 * @param {string} toLang
 * @return {string}
 * @private
 */
function translateWPBakeryAttrs_(shortcode, fromLang, toLang) {
  let result = shortcode;

  for (const attr of WPBAKERY_TRANSLATABLE_ATTRS_) {
    // Паттерн: attr="значение" или attr='значение'
    // Не трогаем значения, содержащие CSS (фигурные скобки)
    const re = new RegExp(`(${attr}=")([^"]*)(")`, "g");
    result = result.replace(re, (match, open, val, close) => {
      // Пропускаем если внутри есть CSS-подобный контент
      if (/[{}]/.test(val)) return match;
      // Пропускаем пустые строки, числа, id-like значения
      if (!val.trim() || /^[\d\-a-z_]+$/.test(val.trim())) return match;
      // Пропускаем значения с URL
      if (/https?:\/\//.test(val)) return match;

      try {
        const translated = LanguageApp.translate(val, fromLang, toLang);
        return open + translated + close;
      } catch (e) {
        return match;
      }
    });

    // То же для одинарных кавычек
    const reSingle = new RegExp(`(${attr}=')([^']*)(')`, "g");
    result = result.replace(reSingle, (match, open, val, close) => {
      if (/[{}]/.test(val)) return match;
      if (!val.trim() || /^[\d\-a-z_]+$/.test(val.trim())) return match;
      if (/https?:\/\//.test(val)) return match;
      try {
        return open + LanguageApp.translate(val, fromLang, toLang) + close;
      } catch (e) {
        return match;
      }
    });
  }

  return result;
}


// ═══════════════════════════════════════════════════════════════════════════
//  ПЕРЕОПРЕДЕЛЯЕМ translateContent_ с учётом WPBakery атрибутов
// ═══════════════════════════════════════════════════════════════════════════

/**
 * Полная версия translateContent_ с поддержкой WPBakery атрибутов.
 * (Перекрывает заглушку выше через переопределение функции ниже)
 * @param {string} content
 * @param {string} fromLang
 * @param {string} toLang
 * @return {string}
 * @private
 */
function translateContent_(content, fromLang, toLang) {

  // ── Шаг 1: токенизация ──────────────────────────────────────────────────
  const tokens = tokenize_(content);

  // ── Шаг 2: собираем текстовые токены ────────────────────────────────────
  const textTokenIndices = [];
  for (let i = 0; i < tokens.length; i++) {
    if (tokens[i].type === "TEXT" && tokens[i].value.trim() !== "") {
      textTokenIndices.push(i);
    }
  }

  // ── Шаг 3: батчевый перевод текстовых токенов ───────────────────────────
  if (textTokenIndices.length > 0) {
    const translatedTexts = batchTranslate_(
      textTokenIndices.map(i => tokens[i].value),
      fromLang,
      toLang
    );
    for (let j = 0; j < textTokenIndices.length; j++) {
      tokens[textTokenIndices[j]].value = translatedTexts[j];
    }
  }

  // ── Шаг 4: переводим атрибуты WPBakery shortcodes ───────────────────────
  const wpbakeryIndices = [];
  for (let i = 0; i < tokens.length; i++) {
    if (tokens[i].type === "WPBAKERY_TAG") {
      wpbakeryIndices.push(i);
    }
  }

  // Собираем значения атрибутов для батчевого перевода
  if (wpbakeryIndices.length > 0) {
    const attrBatches = []; // { tokenIndex, attr, value, quoteChar, matchStart }

    for (const ti of wpbakeryIndices) {
      const sc = tokens[ti].value;
      for (const attr of WPBAKERY_TRANSLATABLE_ATTRS_) {
        // Двойные кавычки
        const reD = new RegExp(`${attr}="([^"]*)"`, "g");
        let m;
        while ((m = reD.exec(sc)) !== null) {
          const val = m[1];
          if (isAttrTranslatable_(val)) {
            attrBatches.push({ tokenIndex: ti, attr, value: val, quoteChar: '"' });
          }
        }
        // Одинарные кавычки
        const reS = new RegExp(`${attr}='([^']*)'`, "g");
        while ((m = reS.exec(sc)) !== null) {
          const val = m[1];
          if (isAttrTranslatable_(val)) {
            attrBatches.push({ tokenIndex: ti, attr, value: val, quoteChar: "'" });
          }
        }
      }
    }

    if (attrBatches.length > 0) {
      const attrTranslated = batchTranslate_(
        attrBatches.map(b => b.value),
        fromLang,
        toLang
      );

      // Подставляем переводы обратно в токены
      for (let k = attrBatches.length - 1; k >= 0; k--) {
        const { tokenIndex, attr, value, quoteChar } = attrBatches[k];
        const translated = attrTranslated[k];
        const q = quoteChar;
        tokens[tokenIndex].value = tokens[tokenIndex].value.replace(
          `${attr}=${q}${value}${q}`,
          `${attr}=${q}${translated}${q}`
        );
      }
    }
  }

  // ── Шаг 5: собираем результат ───────────────────────────────────────────
  return tokens.map(t => t.value).join("");
}


/**
 * Определяет, нужно ли переводить значение атрибута.
 * @param {string} val
 * @return {boolean}
 * @private
 */
function isAttrTranslatable_(val) {
  if (!val || !val.trim()) return false;
  if (/[{}]/.test(val)) return false;         // CSS-контент
  if (/https?:\/\//.test(val)) return false;  // URL
  if (/^[\d\-a-z_:]+$/.test(val.trim())) return false; // id, числа, технические значения
  // Должны быть кириллица или латинские слова с пробелами (не технические строки)
  if (!/[а-яёА-ЯЁa-zA-Zа-яіїєґА-ЯІЇЄҐ]{3,}/.test(val)) return false;
  return true;
}


// ═══════════════════════════════════════════════════════════════════════════
//  УТИЛИТЫ ДЛЯ ОТЛАДКИ (вызываются вручную из редактора)
// ═══════════════════════════════════════════════════════════════════════════

/**
 * Тест токенизатора — запустить вручную в редакторе.
 */
function testTokenizer() {
  const sample = `<strong>Dji Mini 4</strong> - запасний <strong>акумулятор</strong>.
&nbsp;
<!-- обычный шорт код -->
[contact-form-7 id="102c92f" title="button-ques"]
[vc_tta_section title="Революционные материалы" tab_id="1469138165604-165153d8-9412"]
<div id="wrap" class="row tb20 py-[0.2rem]">Текст</div>
<img src="https://example.com/фото-дрона_1.jpg" alt="dji" />`;

  const tokens = tokenize_(sample);
  tokens.forEach((t, i) => Logger.log(`[${i}] ${t.type}: ${JSON.stringify(t.value)}`));
}

/**
 * Интеграционный тест — запустить вручную.
 */
function testTranslate() {
  const sample = `<strong>Новая рама</strong> из магниевого сплава.
[vc_tta_section title="Революционные материалы" tab_id="1469138165604-165153d8-9412"]
<p>Простой текст для перевода.</p>
<!-- не переводить этот комментарий -->
[contact-form-7 id="102c92f" title="кнопка"]
&nbsp;`;

  const result = CUSTOM_TRANSLATE(sample, "ru", "uk");
  Logger.log("=== RESULT ===\n" + result);
}
```

## 📦 Тестовый html № 1 для проверки работы функции CUSTOM_TRANSLATE(A1; "ru"; "uk"; "і")

```html
<strong>Dji Mini 4 intelligent flight battery</strong>- запасной <strong>аккумулятор</strong>для дрона Dji Mini 4.

Батарейка обеспечивает максимальное время полета до 34 минут* (Измерения производились при постоянной скорости 21,6 км/ч в безветренную погоду)

<h3>Захват для DJI Ronin комплект:</h3>
• С-образные ручки - 2 шт.
• монтажные винты - 2 шт.
• кейс сумка - 1 шт.

&nbsp;

<h3>&#x2705; <strong>Кому подойдет</strong></h3>

<hr />

<img class="aligncenter wp-image-22772 size-full" src="https://dronestore.com.ua/wp-content/uploads/2025/10/фото-дрона_1.jpg" alt="dji neo 2" width="730" height="632" />

<!-- обычный шорткод для cf7 -->
[contact-form-7 id="102c92f" title="button-ques"]

<ul>
    <li><strong>DJI Matrice 4T</strong>:
        <ul> 
            <li>Большие инспекционные проекты</li> 
            <li>Профессиональная картография</li> 
            <li>Поиск и спасение с больших высот</li> 
            <li>Там, где требуется лазерный дальномер и гибкость модулей</li> 
        </ul>
    </li>
    <li><strong>DJI Mavic 3T</strong>:
        <ul>
            <li>Мобильность и быстрый запуск</li> 
            <li>Оперативная работа полиции, пожарных</li> 
            <li>Быстрые инспекции крыш, линий электропередач</li> 
            <li>Пользователям, которые хотят что-нибудь профессиональное, но легкое</li>
            <li><span style="color: #ff6600;">Об использовании дронов в зоне боевых действий писать нет смысла и так все знают</span></li>
        </ul>
    </li>
</ul>

<div id="bxr-detail-block-wrap" class="row tb20 py-[0.2rem]">
    <div class="col-lg-12 col-md-12 col-sm-12 col-xs-12">
        <div class="bxr-detail-tab bxr-detail-text" data-tab="detail">DJI Mavic Type-C connector - кабель для соединения пульта управления квадрокоптера DJI Mavic Pro с мобильным устройством посредством порта Reverse Micro USB.</div>
    </div>
</div>
<div id="bxr-additional-info" class="row"></div>

Советуем использовать для зарядки <a href="https://dronestore.com.ua/shop/hab-two-way-charging-hub-dji-mini3-pro/">ХАБ</a>

Рекомендуемое зарядное устройство: зарядное устройство USB-C DJI мощностью 30 Вт или другое зарядное устройство USB Power Delivery.
<h3 style="text-align: center;"><span style="color: #000000;" data-darkreader-inline-color="">Аккумулятор (battery) DJI Mini 4 - купить в Киеве и Украине</span></h3>
<p style="text-align: center;"><span style="color: #000000;" data-darkreader-inline-color="">Купить аккумулятор (батарею) для дрона <strong>Мины 4</strong> в Киеве и Украине в интернет-магазине <a style="color: #000000;" href="https://dronestore.com.ua/uk/" data-darkreader-inline-color="">DroneStore.com.ua</a> - Самый белый выбор дронов в Украине. Гарантия 12 месяцев от <a style="color: #000000;" href="https://www.dji.com/" data-darkreader-inline-color="">производителя</a>. Доставка по Киеву и Украине – Бесплатная.</span></p>
```


## 📦 Тестовый html № 2 для проверки работы функции CUSTOM_TRANSLATE(A1; "ru"; "uk"; "і"), смотрим как работает с тегами WPBakery Page Builder

```html
[vc_row]
    [vc_column css=".vc_custom_1493407566663{padding-top: 100px !important;}"]
        [vc_column_text]
            <h1>DJI Phantom 4 - функции и особенности</h1>
        [/vc_column_text]
        [vc_tta_tour style="flat" shape="square" controls_size="md" active_section="1"]
            [vc_tta_section title="Революционные материалы и улучшеная аэродинамика" tab_id="1469138165604-165153d8-9412"]
                [vc_single_image image="425" img_size="medium" css_animation="left-to-right"]
                [vc_column_text]
                    <div class="wording-area aim-right">
                         <div class="type20-pdp-title image-title text-fix-widow">Только ты, твои мысли и невероятный звук.</div>
                    </div>
                    Новая улучшенная центральная рама из магниевого сплава увеличивает жесткость Phantom 4 без ущерба для веса.
                [/vc_column_text]
            [/vc_tta_section]
            [vc_tta_section title="Настройка пульта" tab_id="1469138890801-2dbd2fcf-b3db"]
                [vc_single_image image="438" img_size="full"]
                [vc_column_text]
                    <h2 class="_2e90 _2e93 _2e95 feature-section-1__title" style="text-align: center;">Простая настройка</h2>
                    Гибкая настройка пульта управления позволяет адаптировать его к вашей манере полета и сделать функционал удобный для вас.
                    <table class=" aligncenter">
						<thead>
							<tr>
								<td><strong>Параметр</strong></td>
								<td><strong>Значення</strong></td>
							</tr>
						</thead>
						<tbody>
							<tr>
								<td><b>Макс. время полета</b></td>
								<td>До 45 минут (в идеальных условиях)</td>
							</tr>
							<tr>
								<td><b>Система передачи</b></td>
								<td>DJI O3 Enterprise (до 8 км CE)</td>
							</tr>
						</tbody>
					</table>
                [/vc_column_text]
            [/vc_tta_section]
        [/vc_tta_tour]
        [vc_video link="https://www.youtube.com/watch?v=JJPSSqMQajA" title="DJI Phantom 4 - Видео обзор"]
        [vc_column_text]
            <h1>Дрон DJI Phantom 4 - Купить в Киеве и Украине</h1>
            <p style="text-align: center;">Бесконечные съемки сценариев. Один гибкое решение.</p>
            <strong>Конструкция: </strong>Карбоновое покрытие ножек, теперь гармонично дополнено магниево-алюминиевым композитом корпуса
            Купить Квадрокоптер DJI Phantom 4 вы можете в интернет магазине <a href="http://dronestore.com.ua">DroneStore</a> - официального дилера DJI.
            Официальный сайт DJI: <a href="http://dji.com">dji.com</a>
        [/vc_column_text]
    [/vc_column]
[/vc_row]
```

