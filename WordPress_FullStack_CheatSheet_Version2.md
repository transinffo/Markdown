# 🏆 WordPress Full Stack Cheat Sheet (6.8+)

**Для разработчика, активно использующего:**  
ACF · Polylang · Yoast SEO · CF7 · Akismet

---

## 💡 Основы PHP для WP

```php
// Получить глобальные переменные
global $post, $wpdb;

// Хуки действия/фильтра
add_action('init', 'my_init');
add_action('wp_enqueue_scripts', 'enqueue_scripts');
add_action('after_setup_theme', 'theme_support');
add_filter('the_content', 'my_content_filter');
add_filter('excerpt_length', function($length){return 20;});
add_action('admin_menu', 'my_admin_menu');

// Получить тип поста
$post_type = get_post_type($post);

// Получить ID текущего пользователя
$current_user = wp_get_current_user();
$user_id = get_current_user_id();

// Проверка прав пользователя
if (current_user_can('edit_posts')) { ... }

// Получить настройки сайта
get_option('blogname');
update_option('my_option', 'value');

// Получить URL медиафайла
$image_url = wp_get_attachment_url($image_id);

// Генерация nonce
$nonce = wp_create_nonce('my_action');

// Проверить nonce
check_admin_referer('my_action');

// Получить ссылку на пост
$link = get_permalink($post);

// Получить родителя поста
$parent_id = wp_get_post_parent_id($post->ID);

// Получить шаблон страницы
$template = get_page_template_slug($post->ID);

// Получить все пользователи
$users = get_users(['role' => 'editor']);

// Получить email администратора
$admin_email = get_option('admin_email');

// Отправить email
wp_mail('me@example.com', 'Тема', 'Текст');
```

---

## 🧩 Работа с ACF (Advanced Custom Fields)

```php
// Получить поле ACF для поста
$field = get_field('custom_field_name', $post->ID);

// Получить группу ACF полей
$fields = get_fields($post->ID);

// Отобразить поле с fallback
echo get_field('subtitle') ?: 'Без подзаголовка';

// Flexible/Repeater поля
if (have_rows('repeater_field')) :
  while (have_rows('repeater_field')) : the_row();
    echo get_sub_field('subfield_name');
  endwhile;
endif;

// Обновить поле
update_field('custom_field', 'Новое значение', $post->ID);

// Проверить наличие поля
if(get_field('acf_field')) { ... }

// Получить ACF поле для пользователя
get_field('profile_field', 'user_'.$user_id);

// Получить ACF поле для термина
get_field('term_field', 'category_'.$term_id);

// Получить image поле
$image = get_field('image_field');
echo wp_get_attachment_image($image['ID'], 'medium');

// Получить файл
$file = get_field('file_field');
echo '<a href="'.$file['url'].'">'.$file['filename'].'</a>';

// Получить галерею
$gallery = get_field('gallery_field');
foreach($gallery as $img) echo wp_get_attachment_image($img['ID'], 'thumbnail');

// Проверить flexible field layout
if(get_row_layout() == 'text_block') { ... }

// Получить дату
$date = get_field('date_field');

// Получить true/false
if(get_field('toggle_field')) { ... }

// Получить post object
$related_post = get_field('related_post');
echo get_the_title($related_post->ID);

// Получить select поле
$select = get_field('select_field');
```

---

## 🌍 Мультиязычность с Polylang

```php
// Получить текущий язык
$current_lang = pll_current_language();

// Получить все языки
$languages = pll_languages_list();

// Ссылка на пост на другом языке
$fr_id = pll_get_post($post->ID, 'fr');
$en_id = pll_get_post($post->ID, 'en');

// Вывести переключатель языков
pll_the_languages(['show_flags' => 1, 'show_names' => 1]);

// Получить slug языка
$lang_slug = pll_get_post_language($post->ID);

// Получить id языка
$lang_id = pll_get_post_language($post->ID, 'slug');

// Получить переводы термина
$term_translations = pll_get_term($term_id, 'fr');

// Получить переводы страницы
$page_ids = pll_get_post_translations($post->ID);

// Получить перевод строки
_e('Пример текста', 'textdomain');
__('Another text', 'textdomain');
_x('text', 'context', 'textdomain');

// Получить список языков для селектора
foreach(pll_languages_list() as $lang) { ... }

// Добавить перевод для строки
pll_register_string('custom_string', 'Default Value');

// Проверить язык на фронте
if(pll_current_language() == 'en') { ... }

// Получить url для языка
$url = pll_home_url('fr');

// Получить url поста для языка
$url = get_permalink(pll_get_post($post->ID, 'en'));

// Получить url термина для языка
$term_url = get_term_link(pll_get_term($term_id, 'en'));

// Получить массив переводов поста
$translations = pll_get_post_translations($post->ID);
```

---

## 🚦 Yoast SEO (метаданные и OpenGraph)

```php
// Получить Yoast SEO title и description
$yoast_title = YoastSEO()->meta->title;
$yoast_desc = YoastSEO()->meta->description;

// Получить канонический url
$canonical_url = YoastSEO()->meta->canonical;

// Получить OpenGraph image
$og_image = YoastSEO()->meta->og_image;

// Получить мета robots
$robots = YoastSEO()->meta->robots;

// Получить breadcrumbs
$breadcrumbs = YoastSEO()->breadcrumbs->breadcrumbs;

// Фильтр для SEO title
add_filter('wpseo_title', function($title) {
  return $title . ' | Мой сайт';
});

// Фильтр для description
add_filter('wpseo_metadesc', function($desc) {
  return $desc . ' - дополнительный текст';
});

// Фильтр для canonical
add_filter('wpseo_canonical', function($url) {
  return $url . '?ref=site';
});

// Фильтр для OpenGraph title
add_filter('wpseo_opengraph_title', function($og_title) {
  return 'Custom OG Title';
});

// Фильтр для OpenGraph description
add_filter('wpseo_opengraph_desc', function($og_desc) {
  return 'Custom OG Description';
});

// Фильтр для OpenGraph image
add_filter('wpseo_opengraph_image', function($img) {
  return 'https://site.com/image.jpg';
});

// Yoast breadcrumbs shortcode
echo do_shortcode('[wpseo_breadcrumb]');

// Yoast indexable check
$is_indexable = YoastSEO()->meta->robots !== 'noindex, nofollow';

// Yoast SEO analysis
$seo_analysis = YoastSEO()->analysis->results;

// Получить primary category
$primary_cat = YoastSEO()->primary_category->get_primary_category($post->ID);

// Получить focus keyword
$keyword = get_post_meta($post->ID, '_yoast_wpseo_focuskw', true);

// Получить SEO score
$score = get_post_meta($post->ID, '_yoast_wpseo_content_score', true);

// Yoast social preview image
$social_img = get_post_meta($post->ID, '_yoast_wpseo_opengraph-image', true);
```

---

## ✉️ Contact Form 7 (CF7)

```php
// Вставить форму в шаблон
echo do_shortcode('[contact-form-7 id="123" title="Контактная форма"]');

// Получить все формы
$forms = WPCF7_ContactForm::find();

// Получить данные формы по ID
$form = WPCF7_ContactForm::get_current();

// JS — успешная отправка
document.addEventListener('wpcf7mailsent', function(event) {
  alert('Спасибо! Форма отправлена.');
}, false);

// JS — ошибка отправки
document.addEventListener('wpcf7invalid', function(event) {
  alert('Ошибка!');
});

// JS — загрузка файлов
document.addEventListener('wpcf7fileadded', function(event) {
  console.log('Файл добавлен!');
});

// JS — сброс формы
document.addEventListener('wpcf7reset', function(event) {
  alert('Форма сброшена');
});

// PHP — успешная отправка
add_action('wpcf7_mail_sent', function($contact_form) {
  // Ваш код
});

// PHP — ошибка отправки
add_action('wpcf7_mail_failed', function($contact_form) {
  // Ваш код
});

// Получить поля формы
$form_fields = $contact_form->scan_form_tags();

// Получить отправленные данные
$submission = WPCF7_Submission::get_instance();
$data = $submission->get_posted_data();

// Добавить custom validation
add_filter('wpcf7_validate_text', 'my_text_validation_filter', 10, 2);
function my_text_validation_filter($result, $tag) {
  // Ваша логика
  return $result;
}

// Custom response
add_action('wpcf7_before_send_mail', 'my_custom_mail');
function my_custom_mail($contact_form) {
  // Ваша логика
}

// Получить status формы
$status = $submission->get_status();

// Получить attachments
$attachments = $submission->uploaded_files();
```

---

## 🛡️ Akismet (Антиспам)

```php
// Проверить комментарий на спам
if (akismet_http_post($comment_data, AKISMET_API_PORT)) {
  // Логика при обнаружении спама
}

// Проверить статус комментария
$comment = get_comment($comment_id);
$is_spam = get_comment_meta($comment->comment_ID, 'akismet_result', true) === 'spam';

// Получить количество заблокированных спамов
$spam_count = get_option('akismet_spam_count');

// Получить статус Akismet
$status = get_option('akismet_api_key');

// Получить мету комментария
$meta = get_comment_meta($comment_id);

// Проверить, включен ли Akismet
$is_enabled = is_plugin_active('akismet/akismet.php');

// Получить все спам-комментарии
$spam_comments = get_comments(['status' => 'spam']);

// Получить оценку Akismet
$akismet_score = get_comment_meta($comment->comment_ID, 'akismet_pro_tip', true);

// Получить историю проверки спама
$history = get_comment_meta($comment->comment_ID, 'akismet_history', true);

// Получить причину спама
$reason = get_comment_meta($comment->comment_ID, 'akismet_error', true);

// Добавить custom проверку
add_filter('akismet_comment_check_response', function($response, $comment) {
  if ($response == 'spam') { /* ... */ }
  return $response;
}, 10, 2);

// Получить статистику Akismet
$stats = akismet_get_stats();

// Получить API key
$api_key = get_option('wordpress_api_key');

// Получить статус ключа
$is_valid = Akismet::verify_key($api_key);
```

---

## 🚀 WP Query & Custom Post Types

```php
// Получить кастомные посты
$args = [
  'post_type' => 'portfolio',
  'posts_per_page' => 5,
  'lang' => pll_current_language()
];
$query = new WP_Query($args);

// Получить все посты
$posts = get_posts(['numberposts' => 10]);

// Получить один пост
$post = get_post($post_id);

// Получить посты с мета-параметром
$args = ['meta_key' => 'featured', 'meta_value' => '1'];
$query = new WP_Query($args);

// Получить посты по таксономии
$args = [
  'post_type' => 'portfolio',
  'tax_query' => [
    ['taxonomy' => 'type', 'field' => 'slug', 'terms' => 'web']
  ]
];
$query = new WP_Query($args);

// Получить посты по автору
$args = ['author' => $user_id];
$query = new WP_Query($args);

// Получить draft посты
$args = ['post_status' => 'draft'];
$query = new WP_Query($args);

// Получить sticky посты
$args = ['post__in' => get_option('sticky_posts')];
$query = new WP_Query($args);

// Получить посты с ACF полем
$args = ['meta_key' => 'acf_field', 'meta_value' => 'value'];
$query = new WP_Query($args);

// Получить посты с датой публикации
$args = ['date_query' => [['after' => '2024-01-01']]];
$query = new WP_Query($args);

// Получить посты с сортировкой
$args = ['orderby' => 'title', 'order' => 'ASC'];
$query = new WP_Query($args);

// Получить посты с пагинацией
$paged = get_query_var('paged') ?: 1;
$args = ['paged' => $paged];
$query = new WP_Query($args);

// Получить посты с поиском
$args = ['s' => 'поиск'];
$query = new WP_Query($args);

// Получить все custom post types
$post_types = get_post_types(['public' => true], 'names');

// Получить custom post type labels
$labels = get_post_type_object('portfolio')->labels;
```

---

## 🏷️ Метаданные и таксономии

```php
// Получить метаполе поста
$meta = get_post_meta($post->ID, 'my_meta_key', true);

// Обновить метаполе
update_post_meta($post->ID, 'my_meta_key', 'value');

// Удалить метаполе
delete_post_meta($post->ID, 'my_meta_key');

// Получить все метаполя
$all_meta = get_post_meta($post->ID);

// Получить метаполе пользователя
$user_meta = get_user_meta($user_id, 'profile_field', true);

// Получить метаполе термина
$term_meta = get_term_meta($term_id, 'meta_key', true);

// Получить таксономии поста
$terms = get_the_terms($post->ID, 'category');

// Получить term slug
$term = get_term_by('slug', 'news', 'category');

// Получить term name
$name = $term->name;

// Получить родительский term
$parent_id = $term->parent;

// Получить все terms таксономии
$terms = get_terms(['taxonomy' => 'category']);

// Получить term link
$link = get_term_link($term);

// Получить term description
$desc = term_description($term_id, 'category');

// Добавить term
wp_insert_term('Новое', 'category');

// Обновить term
wp_update_term($term_id, 'category', ['name' => 'Обновлено']);

// Удалить term
wp_delete_term($term_id, 'category');

// Получить custom taxonomy labels
$labels = get_taxonomy('portfolio_type')->labels;

// Получить terms по meta
$args = [
  'taxonomy' => 'category',
  'meta_query' => [
    ['key' => 'meta_key', 'value' => 'value']
  ]
];
$terms = get_terms($args);
```

---

## 🌐 REST API

```php
// Регистрация маршрута
add_action('rest_api_init', function() {
  register_rest_route('myplugin/v1', '/data/', [
    'methods' => 'GET',
    'callback' => 'my_api_callback'
  ]);
});

function my_api_callback($request) {
  return ['success' => true];
}

// Получение данных через WP_REST_Request
$data = $request->get_param('id');

// Регистрация POST маршрута
register_rest_route('myplugin/v1', '/save/', [
  'methods' => 'POST',
  'callback' => 'save_callback'
]);

// Проверка nonce
check_ajax_referer('wp_rest', '_wpnonce');

// Получить текущего пользователя
$user = wp_get_current_user();

// Получить заголовки запроса
$headers = $request->get_headers();

// Получить параметры запроса
$params = $request->get_params();

// Получить авторизацию
$auth = $request->get_header('authorization');

// Вернуть ошибку
return new WP_Error('error_code', 'Ошибка!', ['status' => 400]);

// Получить JSON ответ
wp_send_json(['ok' => true]);

// Получить REST URL
$rest_url = rest_url('myplugin/v1/data/');

// Получить все маршруты
$routes = rest_get_server()->get_routes();

// Получить коллекцию постов через REST
add_action('rest_api_init', function() {
  register_rest_route('myplugin/v1', '/posts/', [
    'methods' => 'GET',
    'callback' => function() {
      return get_posts(['numberposts' => 5]);
    }
  ]);
});
```

---

## 🖥️ JS для WP + AJAX

```js
// Отправка AJAX-запроса
jQuery.ajax({
  url: my_ajax_object.ajax_url,
  type: 'POST',
  data: { action: 'my_action', foo: 'bar' },
  success: function(res) { console.log(res); }
});

// Получить nonce из объекта
const nonce = my_ajax_object.nonce;

// AJAX с nonce
jQuery.post(my_ajax_object.ajax_url, { action: 'my_action', _ajax_nonce: nonce }, function(res){});

// AJAX для загрузки файла
let form = new FormData();
form.append('action', 'upload_file');
form.append('file', file);
jQuery.ajax({
  url: my_ajax_object.ajax_url,
  type: 'POST',
  data: form,
  contentType: false,
  processData: false,
  success: function(res){ }
});

// AJAX для CF7
document.addEventListener('wpcf7submit', function(e){
  console.log('CF7 отправлено');
});

// AJAX для фильтрации постов
jQuery('#filter').on('change', function() {
  jQuery.post(my_ajax_object.ajax_url, {
    action: 'filter_posts', value: this.value
  }, function(res) { jQuery('#results').html(res); });
});

// AJAX обработчик в PHP
add_action('wp_ajax_my_action', 'my_callback');
add_action('wp_ajax_nopriv_my_action', 'my_callback');
function my_callback() {
  echo json_encode(['success' => true]);
  wp_die();
}

// Получить данные формы
let data = jQuery('#myform').serialize();

// AJAX обновление мета
jQuery.post(my_ajax_object.ajax_url, { action: 'update_meta', post_id: 123, key: 'field', value: 'val' });

// AJAX удаление поста
jQuery.post(my_ajax_object.ajax_url, { action: 'delete_post', id: 123 });

// AJAX получение списка
jQuery.get(my_ajax_object.ajax_url, { action: 'get_list' }, function(res){});

// AJAX для REST API
fetch('/wp-json/myplugin/v1/data')
  .then(res => res.json())
  .then(data => console.log(data));

// AJAX для перевода Polylang
jQuery.post(my_ajax_object.ajax_url, { action: 'get_translation', post_id: 456, lang: 'fr' }, function(res){});
```

---

## 🎨 Полезные хуки и сниппеты

```php
// Добавить класс к body
add_filter('body_class', function($classes) {
  $classes[] = 'custom-class';
  return $classes;
});

// Кастомная страница шаблона
if (is_page('about')) { /* ... */ }

// Добавить свойство в REST API ответ
add_filter('rest_prepare_post', function($response, $post, $request){
  $response->data['acf'] = get_fields($post->ID);
  return $response;
}, 10, 3);

// Добавить custom поле в WP_User
add_action('show_user_profile', function($user){
  echo '<input type="text" name="custom_field">';
});

// Добавить meta box для поста
add_action('add_meta_boxes', function(){
  add_meta_box('custom_box', 'Мета', 'custom_box_callback', 'post');
});
function custom_box_callback($post){
  echo '<input type="text">';
}

// Изменить title страницы
add_filter('pre_get_document_title', function($title){
  return 'Custom Title';
});

// Очистить кэш
wp_cache_flush();

// Кастомный rewrite rule
add_action('init', function(){
  add_rewrite_rule('^custom-url/?', 'index.php?page_id=2', 'top');
});

// Добавить shortcode
add_shortcode('my_shortcode', function($atts){
  return 'Content';
});

// Удалить emoji script
remove_action('wp_head', 'print_emoji_detection_script', 7);

// Удалить jQuery migrate
add_filter('wp_default_scripts', function($scripts){
  if(!is_admin() && $scripts->registered['jquery']){
    $scripts->registered['jquery']->deps = array_diff($scripts->registered['jquery']->deps, ['jquery-migrate']);
  }
});

// Добавить SVG поддержку
add_filter('upload_mimes', function($mimes){
  $mimes['svg'] = 'image/svg+xml';
  return $mimes;
});

// Отключить Gutenberg
add_filter('use_block_editor_for_post', '__return_false');

// Скрыть admin bar
add_filter('show_admin_bar', '__return_false');
```

---

## ⚡ Советы

- Работайте через дочерние темы/кастомные плагины.
- Всегда используйте nonce для безопасности AJAX.
- Для мультиязычных ACF полей — используйте ключи для языков.
- Yoast SEO: не меняйте мета-теги напрямую, используйте фильтры.
- Для REST API — проверяйте права доступа.
- Используйте кэширование для сложных запросов.
- Проверяйте совместимость плагинов при обновлениях.
- Старайтесь не изменять ядро WP.
- Используйте WP_DEBUG=true для отладки.
- Оформляйте кастомные поля и таксономии с помощью register_*
- Для сложных форм — интеграция с CF7 и AJAX.
- Для SEO — настраивайте sitemap и robots через Yoast.
- В ACF используйте flexible content для сложных блоков.
- Для мультиязычных проектов — связывайте записи через Polylang API.
- Для антиспама — интеграция Akismet + custom фильтры.
- Для кастомных REST API — документируйте эндпоинты и права.
- Используйте WP Cron для задач по расписанию.
- Для производительности — оптимизируйте SQL-запросы и индексы.
- Не храните секреты в коде (используйте wp-config).
- Поддерживайте чистую структуру тем и плагинов.
- Документируйте все кастомные функции/эндпоинты.

---

*Шпаргалка для WP 6.8+ · Полиглот · SEO-гуру · Full Stack 🚀*