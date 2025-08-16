# 🌟 WordPress Cheat Sheet

---

## ✅  Отзывы Google

### Создаем группу полей "Отзывы Google" с полями:

1. Поле 1 - Файл "google_reviews_json" - Файл (массив-все)
2. Поле 2 - Выбор отзывов "selected_reviews" - выбор (select) - выбрать несколько - да
3. Назначаем в каких шаблонах выводить

### Код для functions.php

```php
//Отзывы Google
add_filter('acf/load_field/name=selected_reviews', function ($field) {
    // Получаем значение поля 'google_reviews_json' из текущего поста
    $post_id = get_the_ID();
    $file = get_field('google_reviews_json', $post_id);

    if ($file && isset($file['url'])) {
        $json = file_get_contents($file['url']);
        $data = json_decode($json, true);

        if (is_array($data) && isset($data['reviews'])) {
            $choices = [];
            foreach ($data['reviews'] as $review) {
                $id    = $review['id'];
                $stars = isset($review['stars']) ? str_repeat('★', (int)$review['stars']) : '';
                $date  = isset($review['date']) ? $review['date'] : '';
                $text  = mb_strimwidth($review['text'], 0, 180, '...');

                $label = trim("{$stars} · {$date} · {$review['authorName']} — {$text}");
                $choices[$id] = $label;
            }
            $field['choices'] = $choices;
        }
    }

    return $field;
});

// Шорткод: [selected_google_reviews]
add_shortcode('selected_google_reviews', function () {
    $post_id = get_the_ID();
    $json_file = get_field('google_reviews_json', $post_id);
    $selected_ids = get_field('selected_reviews', $post_id);

    if (!$json_file || !$selected_ids || !isset($json_file['url'])) {
        return '<p><em>Отзывы не выбраны или файл не найден.</em></p>';
    }

    $json = file_get_contents($json_file['url']);
    $data = json_decode($json, true);

    if (!is_array($data) || !isset($data['reviews'])) {
        return '<p><em>Ошибка чтения отзывов.</em></p>';
    }

    $reviews = $data['reviews'];

    // Преобразуем в ассоциативный массив для быстрого доступа по ID
    $indexed = [];
    foreach ($reviews as $r) {
        $indexed[$r['id']] = $r;
    }

    // URL Google страницы
    $google_url = isset($data['pageUrl']) ? $data['pageUrl'] : '';

    // Значения рейтинга и количества отзывов
    $ratingValue = isset($data['ratingValue']) ? $data['ratingValue'] : 0;
    $reviewCount = isset($data['reviewCount']) ? $data['reviewCount'] : 0;

    // Определим язык и заголовки
    $lang = function_exists('pll_current_language') ? pll_current_language() : 'uk';
    $title = ($lang === 'uk') ? 'Відгуки про нас на Google' : 'Отзывы о нас на Google';
    $link_text = ($lang === 'uk') ? 'Подивитись усі відгуки' : 'Посмотреть все отзывы';

    $output = '<div class="selected-google-reviews">';
    $output .= '<h3 class="google-reviews-title">' . esc_html($title) . '</h3>';
    $output .= '<div class="google-reviews">';

    $schema_reviews = []; // Сюда соберем микроразметку

    foreach ($selected_ids as $id) {
        if (!isset($indexed[$id])) continue;
        $r = $indexed[$id];

        $avatar = esc_url($r['avatar']);
        $author = esc_html($r['authorName']);
        $stars  = (int) $r['stars'];
        $date   = esc_html($r['date']);
        $text   = esc_html($r['text']);

        // HTML отзыв
        $output .= '<div class="google-review" style="margin-bottom:20px;padding:5px;">';
        $output .= '<p style="display:flex;align-items:center;width:100%;"><img src="' . $avatar . '" alt="' . $author . '" width="36" height="36" style="border-radius:50%; vertical-align:middle; margin-right:8px;"> <strong>' . $author . '</strong></p>';
        $output .= '<p style="margin: 2px 0;width:100%;"><span style="color:#f39c12;">' . str_repeat('★', $stars) . '</span> · ' . $date . '</p>';
        $output .= '<p style="margin-top: 5px;max-height:200px;overflow-y:auto;">' . $text . '</p>';
        $output .= '</div>';

        // JSON-LD для отзыва
        $schema_reviews[] = [
            "@type" => "Review",
            "author" => [
                "@type" => "Person",
                "name" => $author
            ],
            "reviewRating" => [
                "@type" => "Rating",
                "ratingValue" => $stars,
                "bestRating" => 5,
                "worstRating" => 1
            ],
            "reviewBody" => $text,
            "datePublished" => $date 
        ];
    }

    $output .= '</div>'; // .google-reviews

    // Ссылка на Google
    $output .= '<h3 style="margin:30px 0;text-align:center;"><a href="' . esc_url($google_url) . '" target="_blank">' . esc_html($link_text) . '</a></h3>';
    $output .= '</div>'; // .selected-google-reviews

    // JSON-LD микроразметка
    if (!empty($schema_reviews)) {
        $organization = [
            "@context" => "https://schema.org",
            "@type" => "Organization",
            "name" => get_bloginfo('name'),
            "aggregateRating" => [
                "@type" => "AggregateRating",
                "ratingValue" => $ratingValue,
                "reviewCount" => $reviewCount
            ],
            "review" => $schema_reviews
        ];

        $output .= '<script type="application/ld+json">' . wp_json_encode($organization, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES | JSON_PRETTY_PRINT) . '</script>';
    }

    return $output;
});

//Показываем шорткод в админке под полем selected_reviews (в записи)
add_action('acf/render_field/name=selected_reviews', function ($field) {
    echo '<div style="margin-top:10px; background: #f9f9f9; padding:10px; border-left: 4px solid #0073aa;">';
    echo '<strong>📋 Шорткод для вставки:</strong><br>';
    echo '<code style="font-size: 16px;">[selected_google_reviews]</code>';
    echo '<p style="margin: 5px 0 0; font-size: 13px;">Выберите файл с отзывами, сохраните страницу, выберите отзывы, вставьте этот шорткод в контент и опять сохраните страницу.</p>';
    echo '</div>';
});

//разрешить application/json для ACF поля «Файл»
add_filter('acf/fields/file/query', function ($args, $field, $post_id) {
    // Разрешаем json-файлы для ACF поля
    $args['post_mime_type'] = ['application/json'];
    return $args;
}, 10, 3);

// увеличил высоту select поля selected_reviews в админке
function my_acf_admin_styles() {
    echo '<style>
        div[data-name="selected_reviews"] select
        {
            height: 300px !important;
        }
    </style>';
}
add_action('admin_head', 'my_acf_admin_styles');
```

---

