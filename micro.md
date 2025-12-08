# 🌈 Микроразметка


## 📦 Микроразметка для google reviews

```js
<script type="application/ld+json">{
    "@context": "https://schema.org",
    "@type": "Organization",
    "name": "Somatica",
    "aggregateRating": {
        "@type": "AggregateRating",
        "ratingValue": 4.7,
        "reviewCount": 75
    },
    "review": [
        {
            "@type": "Review",
            "author": {
                "@type": "Person",
                "name": "Олег Кучма"
            },
            "reviewRating": {
                "@type": "Rating",
                "ratingValue": 5,
                "bestRating": 5,
                "worstRating": 1
            },
            "reviewBody": "Проходил реабилитацию после серьезной травмы позвоночника. Очень доволен подходом клиники Somatica. Специалисты подобрали индивидуальную программу и уже после нескольких сеансов появился прогресс. Спасибо команде за профессионализм и поддержку! Теперь чувствую себя гораздо лучше.",
            "datePublished": "28.10.24"
        },
        {
            "@type": "Review",
            "author": {
                "@type": "Person",
                "name": "ohrana.solsif"
            },
            "reviewRating": {
                "@type": "Rating",
                "ratingValue": 5,
                "bestRating": 5,
                "worstRating": 1
            },
            "reviewBody": "Очень доволен лечением в somatica.\nПосле длительных болей в коленях обратился в эту клинику, и врачи помогли мне вернуться к активной жизни.",
            "datePublished": "28.10.24"
        },
        {
            "@type": "Review",
            "author": {
                "@type": "Person",
                "name": "Леонид"
            },
            "reviewRating": {
                "@type": "Rating",
                "ratingValue": 5,
                "bestRating": 5,
                "worstRating": 1
            },
            "reviewBody": "Клиника SOMATICA помогла мне решить хронические проблемы с суставами\nВрачи очень внимательные назначили курс реабилитации, после которого я почувствовал значительное улучшение.\nРекомендую эту клинику всем, кто ищет профессиональную помощь",
            "datePublished": "21.10.24"
        }
    ]
}</script>
```

---

## 📦 Вывод микроразметки google reviews (виджет сервиса sociablekit.com )

### Выводим сами отзывы таким кодом:

```html
<div class="sk-ww-google-reviews" data-embed-id="25630118"></div><script src="https://widgets.sociablekit.com/google-reviews/widget.js" defer></script>
```

---

### Вставляем в php (чистый php) содержимое data-embed-id : 

```php
<?php

$json_url = 'https://data.accentapi.com/feed/25630118.json';

$json_data = file_get_contents($json_url);
if (!$json_data) exit;

$data = json_decode($json_data);
if (!$data) exit;

// Данные для Organization
$org_name = $data->google_data_structure_json->itemReviewed->name ?? '';
$rating_value = $data->google_data_structure_json->ratingValue ?? '0';
$review_count = $data->google_data_structure_json->reviewCount ?? '0';

// Формируем массив отзывов с фильтром rating >= 4 и ограничением 10
$reviews = [];
$max_reviews = 10;

if (!empty($data->reviews) && is_array($data->reviews)) {
    foreach ($data->reviews as $review) {
        $rating = $review->rating ?? '0';
        if ((float)$rating >= 4) {
            $reviews[] = [
                "@type" => "Review",
                "author" => [
                    "@type" => "Person",
                    "name" => $review->reviewer_name ?? ''
                ],
                "reviewRating" => [
                    "@type" => "Rating",
                    "ratingValue" => $rating, // оставляем как текст
                    "bestRating" => "5",
                    "worstRating" => "1"
                ],
                "reviewBody" => isset($review->review_text) ? strip_tags($review->review_text) : '',
                "datePublished" => isset($review->review_date_time) ? date('d.m.y', strtotime($review->review_date_time)) : ''
            ];

            if (count($reviews) >= $max_reviews) break; // ограничиваем до 10 отзывов
        }
    }
}

// Финальный массив LD+JSON
$ld_json = [
    "@context" => "https://schema.org",
    "@type" => "Organization",
    "name" => $org_name,
    "aggregateRating" => [
        "@type" => "AggregateRating",
        "ratingValue" => $rating_value, // как текст
        "reviewCount" => $review_count  // как текст
    ],
    "review" => $reviews
];

echo '<script type="application/ld+json">' . json_encode($ld_json, JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT) . '</script>';
?>
```

---

### Вставляем в php (на Joomla API) содержимое data-embed-id и выводим только на 2 страницах: 

```php
<?php
defined('_JEXEC') or die;

// Получаем путь текущей страницы (без домена)
$current_path = JUri::getInstance()->getPath();

// Список путей, на которых нужно выводить микроразметку
$allowed_paths = [
    '/ua/',
    '/ua/vidhuky.html'
];

if (!in_array($current_path, $allowed_paths)) {
    return; // не выводим на других страницах
}

// URL JSON
$json_url = 'https://data.accentapi.com/feed/25630118.json';

// Получаем JSON через Joomla API
$http = JHttpFactory::getHttp();
try {
    $response = $http->get($json_url);
    $json_data = $response->body;
} catch (Exception $e) {
    return; // не удалось получить данные
}

$data = json_decode($json_data);
if (!$data) return;

// Данные для Organization
$org_name = $data->google_data_structure_json->itemReviewed->name ?? '';
$rating_value = $data->google_data_structure_json->ratingValue ?? '0';
$review_count = $data->google_data_structure_json->reviewCount ?? '0';

// Формируем массив отзывов с фильтром rating >= 4 и лимитом 10
$reviews = [];
$max_reviews = 10;

if (!empty($data->reviews) && is_array($data->reviews)) {
    foreach ($data->reviews as $review) {
        $rating = $review->rating ?? '0';
        if ((float)$rating >= 4) {
            $reviews[] = [
                "@type" => "Review",
                "author" => [
                    "@type" => "Person",
                    "name" => $review->reviewer_name ?? ''
                ],
                "reviewRating" => [
                    "@type" => "Rating",
                    "ratingValue" => $rating,
                    "bestRating" => "5",
                    "worstRating" => "1"
                ],
                "reviewBody" => isset($review->review_text) ? strip_tags($review->review_text) : '',
                "datePublished" => isset($review->review_date_time) ? date('d.m.y', strtotime($review->review_date_time)) : ''
            ];

            if (count($reviews) >= $max_reviews) break;
        }
    }
}

// Формируем LD+JSON
$ld_json = [
    "@context" => "https://schema.org",
    "@type" => "Organization",
    "name" => $org_name,
    "aggregateRating" => [
        "@type" => "AggregateRating",
        "ratingValue" => $rating_value,
        "reviewCount" => $review_count
    ],
    "review" => $reviews
];

echo '<script type="application/ld+json">' . json_encode($ld_json, JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT) . '</script>';
?>
```

---

