
# 🌈 Apps scripts для гугл таблиц


## 📦 Функция-переводчик контента, умеет работать c шорткодами и html. Пример: =CUSTOM_TRANSLATE(A1;'ru';'uk', strong)

```text
/**
 * Универсальный переводчик для WordPress (WP-Shortcodes + HTML).
 * Порядок аргументов: текст, источник, цель, принудительно.
 */
function CUSTOM_TRANSLATE(text, sourceLang, targetLang, force) {
  if (!text || typeof text !== 'string') return text;
  
  const source = sourceLang ? sourceLang.toLowerCase() : "";
  const target = targetLang ? targetLang.toLowerCase() : "uk";

  try {
    // 1. Лингвистический фильтр (і/ї/є для UK, ы/э/ъ/ё для RU)
    if (!force) {
      var ukrSpecs = /[ііїєєґҐ]/i;
      var rusSpecs = /[ыэъёЫЭЪЁ]/i;

      if (target === "uk" && ukrSpecs.test(text)) return text;
      if (target === "ru" && rusSpecs.test(text)) return text;
    }

    // 2. Маскировка структуры
    var placeholders = {};
    var index = 0;
    var regex = /(<\/?[^>]+>|\[\/?[^\]]+\]|https?:\/\/[^\s"']+|<[^>]+>|&[a-z#0-9]+;|<!--[\s\S]*?-->)/g;

    var maskedText = text.replace(regex, function(match) {
      var key = "TRNS" + index;
      placeholders[key] = match;
      index++;
      return " " + key + " "; 
    });

    if (!maskedText.replace(/TRNS\d+/g, '').trim()) return text;

    // 3. Перевод
    var translatedText = LanguageApp.translate(maskedText, source, target);

    // 4. Восстановление
    var restoredText = translatedText;
    var keys = Object.keys(placeholders).sort((a, b) => {
      return parseInt(b.replace("TRNS", "")) - parseInt(a.replace("TRNS", ""));
    });

    keys.forEach(function(key) {
      var searchRegex = new RegExp("\\b" + key + "\\b", "gi");
      restoredText = restoredText.replace(searchRegex, placeholders[key]);
    });

    // 5. Очистка форматирования (Финальная версия)
    return restoredText
      .replace(/(\]|\>)\s+(?!\s)/g, '$1') // Убираем пробел после > если там текст
      .replace(/(?!\s)\s+(\[|\<)/g, '$1') // Убираем пробел перед < если там текст
      .replace(/\s+(<\/a>)/g, '$1')       // Убираем пробел перед </a>
      .replace(/lang="(ru|uk|en)"/g, 'lang="' + target + '"') // Автозамена атрибута lang
      .replace(/(\/>|<\/img>)([А-Яа-яІіЇїЄє])/g, '$1\n\n$2') // Добавляем перенос после фото
      .replace(/\[\s+/g, '[')
      .replace(/\s+\]/g, ']')
      .replace(/<\s+\//g, '</')
      .replace(/<!--\s*more\s*-->/gi, '<!--more-->')
      .replace(/[\x00-\x1F\x7F]/g, '') // Удаляет скрытые управляющие символы, включая <0x01>
      .trim();

  } catch (e) {
    return "Error: " + e.message;
  }
}
```
