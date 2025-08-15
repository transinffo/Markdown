# 🌟 WordPress Cheat Sheet

---

## ✅  Отзывы Google

### Код для functions.php

```php
//acf/load_field — заполняем select отзывами (работает и в постах)
              add_filter('acf/load_field/name=selected_reviews', function ($field) {
                  // Получаем значение поля 'google_reviews_json' из текущего поста
                  $post_id = get_the_ID();
                  $file = get_field('google_reviews_json', $post_id);

                  if ($file && isset($file['url'])) {
                      $json = file_get_contents($file['url']);
                      $reviews = json_decode($json, true);

                      if (is_array($reviews)) {
                          $choices = [];
                          foreach ($reviews as $review) {
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
                  $reviews = json_decode($json, true);

                  if (!is_array($reviews)) {
                      return '<p><em>Ошибка чтения отзывов.</em></p>';
                  }

                  // Преобразуем в ассоциативный массив для быстрого доступа по ID
                  $indexed = [];
                  foreach ($reviews as $r) {
                      $indexed[$r['id']] = $r;
                  }

                  // Определим язык и заголовки
                  $lang = function_exists('pll_current_language') ? pll_current_language() : 'uk';
                  $title = ($lang === 'uk') ? 'Відгуки про нас на Google' : 'Отзывы о нас на Google';
                  $link_text = ($lang === 'uk') ? 'Подивитись усі відгуки' : 'Посмотреть все отзывы';

                  $google_url = 'https://www.google.com.ua/maps/place/%D0%A3%D0%BA%D1%80%D0%B0%D1%97%D0%BD%D1%81%D1%8C%D0%BA%D0%B8%D0%B9+%D1%86%D0%B5%D0%BD%D1%82%D1%80+%D1%80%D0%B5%D0%B0%D0%B1%D1%96%D0%BB%D1%96%D1%82%D0%B0%D1%86%D1%96%D1%97+SOMATICA/@50.4244519,30.4596705,14.25z/data=!4m8!3m7!1s0x40d4cfe18e74597f:0x69c07d2df5d1faf6!8m2!3d50.423127!4d30.4665172!9m1!1b1!16s%2Fg%2F11l6ll64q3?hl=' . $lang . '&entry=ttu';

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
                              "ratingValue" => 4.7,    // здесь можешь подставить реальное значение средней оценки
                              "reviewCount" => 75      // здесь — количество отзывов
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
                  echo '<p style="margin: 5px 0 0; font-size: 13px;">Вставьте этот шорткод в контент этой записи, чтобы вывести отзывы.</p>';
                  echo '</div>';
              });


              //разрешить application/json для ACF поля «Файл»
              add_filter('acf/fields/file/query', function ($args, $field, $post_id) {
                  // Разрешаем json-файлы для ACF поля
                  $args['post_mime_type'] = ['application/json'];
                  return $args;
              }, 10, 3);
```

---

