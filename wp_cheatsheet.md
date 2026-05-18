# 🌟 WordPress Cheat Sheet

## ✅  Выполнить в консоли браузера js для получения sql запроса со списком id категориий или брендов на странице
```js
'SELECT * FROM `wp_terms` WHERE `term_id` IN (' +
[...document.querySelectorAll('tr[id^="tag-"]')]
  .map(el => el.id.replace('tag-', ''))
  .join(', ') +
') LIMIT 0, 500;'
```


## ✅  Выполнить в консоли браузера js для получения sql запроса со списком id постов на странице
```js
'SELECT * FROM `wp_posts` WHERE `ID` IN (' +
[...document.querySelectorAll('tr[id^="post-"]')]
  .map(el => el.id.replace('post-', ''))
  .join(', ') +
') LIMIT 0, 500;'
```

## ✅  Программно кликаем по кнопке Обновить на странице списка товаров woocommerce
```js
(async () => {
    // Находим все кнопки "Свойства" (Quick Edit) на странице товаров
    const editButtons = document.querySelectorAll('#the-list .editinline');
    
    console.log(`Найдено товаров: ${editButtons.length}. Начинаю процесс обновления...`);

    for (let i = 0; i < editButtons.length; i++) {
        const btn = editButtons[i];
        
        // Находим родительскую строку <tr>, у товаров ID вида "post-18859"
        const row = btn.closest('tr');
        if (!row) continue;
        
        const postId = row.id.replace('post-', ''); 
        
        console.log(`[${i + 1}/${editButtons.length}] Обработка товара ID: ${postId}`);

        // 1. Открываем "Свойства"
        btn.click();

        // 2. Ждем отрисовки формы (для товаров форма может быть тяжелее, ставим небольшую задержку)
        // Если интернет быстрый, можно уменьшить до 1000-1500
        await new Promise(r => setTimeout(r, 2000));

        // 3. Ищем строку редактирования по ID "edit-18859"
        const editRow = document.getElementById(`edit-${postId}`);
        if (editRow) {
            // В форме товара кнопка сохранения обычно имеет классы .save и .button-primary
            const saveBtn = editRow.querySelector('button.save');
            
            if (saveBtn) {
                saveBtn.click();
                console.log(`✅ Товар ${postId} отправлен на обновление`);
            } else {
                console.error(`❌ Кнопка сохранения не найдена в форме #edit-${postId}`);
            }
        } else {
            console.error(`❌ Форма Quick Edit для товара #edit-${postId} не появилась`);
        }

        // 4. Ждем завершения AJAX-запроса перед переходом к следующему товару
        // Это важно, чтобы WooCommerce успел обновить мета-данные в БД
        await new Promise(r => setTimeout(r, 2000));
    }

    console.log('--- Все товары на этой странице обработаны! ---');
})();
```


## ✅  Программно кликаем по кнопке Обновить на странице списка кастомного бренда или категорий товаров woocommerce
```js
(async () => {
    // Находим все кнопки "Свойства"
    const editButtons = document.querySelectorAll('#the-list .editinline');
    
    console.log(`Найдено элементов: ${editButtons.length}. Начинаю процесс...`);

    for (let i = 0; i < editButtons.length; i++) {
        const btn = editButtons[i];
        
        // Находим родительскую строку <tr>, чтобы вытащить ID (например, "tag-248")
        const row = btn.closest('tr');
        if (!row) continue;
        
        const tagId = row.id.replace('tag-', ''); // Получаем чистый ID (например, 248)
        
        console.log(`[${i + 1}/${editButtons.length}] Обработка ID: ${tagId}`);

        // 1. Кликаем "Свойства"
        btn.click();

        // 2. Ждем, пока WP вставит форму в DOM (ID формы будет "edit-248")
        await new Promise(r => setTimeout(r, 5000));

        // 3. Ищем кнопку сохранения внутри появившейся строки редактирования
        const editRow = document.getElementById(`edit-${tagId}`);
        if (editRow) {
            const saveBtn = editRow.querySelector('button.save');
            if (saveBtn) {
                saveBtn.click();
                console.log(`✅ Нажато "Обновить" для ID ${tagId}`);
            } else {
                console.error(`❌ Кнопка .save не найдена внутри #edit-${tagId}`);
            }
        } else {
            console.error(`❌ Форма редактирования #edit-${tagId} не появилась`);
        }

        // 4. Ждем завершения AJAX-запроса (строка должна обновиться)
        // Ставим 1.5 - 2 секунды, чтобы сервер не "лег" от пачки запросов
        await new Promise(r => setTimeout(r, 1500));
    }

    console.log('--- Все категории обработаны! ---');
})();
```


## ✅  Вывод swiper слайдера (пример с разными размерами фото)

### Скрипты тут:
```html
C:\Users\trans\work\CMS\swiper
```
или тут:
```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/swiper@12/swiper-bundle.min.css"
/>
<script src="https://cdn.jsdelivr.net/npm/swiper@12/swiper-bundle.min.js"></script>
```

### Подключение в теме:
```php
wp_enqueue_style( 'theme-swiper-css', get_template_directory_uri() . '/assets/css/swiper-bundle.min.css');
wp_enqueue_script( 'theme-swiper-js', get_template_directory_uri() . '/assets/js/swiper-bundle.min.js', array(), '20201008', false );
```

### html + css + js :
```html
<div class="single_article_last_posts swiper">
	<!-- Additional required wrapper -->
	<div class="swiper-wrapper">
		<!-- 1 slide -->
		<div class="swiper-slide">
			<a href="/"> <img src="/" alt="">
				<div class="slide_title">Title</div>
			</a>
		</div>
		<!-- 1 slide -->
	</div>
	<!-- If we need pagination -->
	<div class="swiper-pagination"></div>
	<!-- If we need navigation buttons -->
	<div class="swiper-button-prev"></div>
	<div class="swiper-button-next"></div>
</div>
```

```css
.single_article_last_posts.swiper {
  max-width: 787px;
  padding: 0 50px;
}

.single_article .swiper-slide img {
    width: 100%;
    height: 200px;
    object-fit: cover; 
    border-radius: 10px;
}

.slide_title {
    text-align: center;
    font-size: 16px;
    color: #0098b0;
    margin: 10px 0;
}


.swiper-pagination.swiper-pagination-bullets.swiper-pagination-horizontal{
    position: relative;
    margin-top: 10px;
}

.swiper-button-next, .swiper-button-prev{
    color: #00b0b0;
}
```

```js
  const swiper = new Swiper('.single_article_last_posts.swiper', {
  slidesPerView: 3,
  spaceBetween: 50,
  slidesPerGroup: 1,
  loop: true,

  pagination: {
    el: '.swiper-pagination',
  },

  navigation: {
    nextEl: '.swiper-button-next',
    prevEl: '.swiper-button-prev',
  },

  breakpoints: {
            0:    { slidesPerView: 1 },
            768:  { slidesPerView: 2 },
            1024: { slidesPerView: 3 }
   }

});
```





## ✅  Добавляет CSS-класс в <body>, соответствующий активному шаблону категории.

### Кастомные шаблоны должны лежать в /template-parts, например:

/theme/template-parts/template-analizy-i-ceny.php

### А acf поля (да\нет) должны быть вида:

is_template_analizy_i_ceny

### Новый код для archive.php:
```php
<?php
/**
 * Универсальный шаблон архива категорий
 */

get_header();

if ( is_category() ) {
    $category = get_queried_object();
    $fields = get_fields( $category );

    if ( $fields && is_array( $fields ) ) {
        foreach ( $fields as $field_key => $field_value ) {
            // Ищем поля вида is_template_*
            if ( strpos( $field_key, 'is_template_' ) === 0 && $field_value ) {
                // Преобразуем ключ в имя файла шаблона
                // Пример: is_template_analizy_i_ceny_group → template-analizy-i-ceny-group.php
                $template_slug = str_replace( '_', '-', substr( $field_key, strlen( 'is_template_' ) ) );
                $template_file = __DIR__ . '/template-parts/template-' . $template_slug . '.php';

                if ( file_exists( $template_file ) ) {
                    include_once $template_file;
                    get_footer();
                    exit;
                }
            }
        }
    }

    // Если ничего не найдено — шаблон по умолчанию
    include_once __DIR__ . '/archive-default.php';
}

get_footer();
```
### Добавляем в functions.php:
```php
/**
 * Добавляет CSS-класс в <body>, соответствующий активному шаблону категории.
 */
function qlab_custom_category_body_class( $classes ) {
    if ( is_category() ) {
        $category = get_queried_object();
        $fields = get_fields( $category );

        if ( $fields && is_array( $fields ) ) {
            foreach ( $fields as $field_key => $field_value ) {
                if ( strpos( $field_key, 'is_template_' ) === 0 && $field_value ) {
                    $template_slug = 'template-' . str_replace( '_', '-', substr( $field_key, strlen( 'is_template_' ) ) );
                    $classes[] = sanitize_html_class( $template_slug );
                    return $classes; // сразу выходим, чтобы не добавлять другие
                }
            }
        }

        // По умолчанию
        $classes[] = 'template-archive-default';
    }

    return $classes;
}
add_filter( 'body_class', 'qlab_custom_category_body_class' );
```
---

## ✅  сделать копии постов на русском языке для Polylang (язык контента по умолчанию украинский)
```php
// код для functions.php
// Клонируем пост(ы) на украинском языке на русский с Polylang
// https://site.com/wp-admin/?clone_post=2315 - клонируем пост с ID 2315
// https://site.com/wp-admin/?clone_post=all  - клонируем все посты

add_action('admin_init', function () {
    if (!current_user_can('manage_options')) return;
    if (!isset($_GET['clone_post'])) return;

    // Проверка наличия Polylang
    if (!function_exists('pll_get_post_language') || !function_exists('pll_save_post_translations')) {
        echo "Polylang не активирован или отсутствуют функции Polylang.";
        exit;
    }

    $clone_param = $_GET['clone_post'];
    $post_ids_to_clone = [];

    // Если параметр 'all', получаем все посты на украинском
    if ($clone_param === 'all') {
        $uk_posts = new WP_Query([
            'post_type'      => 'any',
            'lang'           => 'uk',
            'posts_per_page' => -1,
            'fields'         => 'ids',
            'no_found_rows'  => true,
        ]);
        $post_ids_to_clone = $uk_posts->posts;
    } else {
        // Иначе, клонируем только один пост по ID
        $src_id = intval($clone_param);
        if ($src_id > 0) {
            $post_ids_to_clone[] = $src_id;
        }
    }

    if (empty($post_ids_to_clone)) {
        echo "Посты для клонирования не найдены.";
        exit;
    }

    $cloned_count = 0;
    $log_messages = [];

    foreach ($post_ids_to_clone as $src_id) {
        $src = get_post($src_id);
        if (!$src) {
            $log_messages[] = "Источник (ID {$src_id}) не найден.";
            continue;
        }

        $src_lang = pll_get_post_language($src_id);
        if ($src_lang !== 'uk') {
            $log_messages[] = "Источник (ID {$src_id}) не помечен как украинский (lang = {$src_lang}).";
            continue;
        }

        // Проверяем, есть ли уже перевод
        $existing = pll_get_post_translations($src_id);
        if (!empty($existing['ru'])) {
            $log_messages[] = "У исходного поста (ID {$src_id}) уже есть русская версия (ID: {$existing['ru']}).";
            continue;
        }

        // создаём пост-клон
        $new_id = wp_insert_post([
            'post_title'     => $src->post_title,
            'post_content'   => $src->post_content,
            'post_excerpt'   => $src->post_excerpt,
            'post_status'    => $src->post_status,
            'post_type'      => $src->post_type,
            'post_author'    => $src->post_author,
            'post_parent'    => $src->post_parent,
            'menu_order'     => $src->menu_order,
            'comment_status' => $src->comment_status,
            'ping_status'    => $src->ping_status,
        ]);

        if (!$new_id) {
            $log_messages[] = "Не удалось создать пост для ID {$src_id}.";
            continue;
        }

        // featured image
        $thumb_id = get_post_thumbnail_id($src_id);
        if ($thumb_id) {
            set_post_thumbnail($new_id, $thumb_id);
            update_post_meta($new_id, '_thumbnail_id', $thumb_id);
        }

        // Список ACF полей
        $acf_keys = [
            'product_stock',
            'product_sale',
            'product_short_desc',
            'product_desc',
            'product_charact',
        ];

        // специальные мета
        $special_meta = [
            '_product_image_gallery',
            '_wp_page_template',
            '_thumbnail_id',
        ];

        // Копируем указанные ACF поля + их мета-ключи
        foreach ($acf_keys as $k) {
            $vals = get_post_meta($src_id, $k);
            if ($vals) {
                delete_post_meta($new_id, $k);
                foreach ($vals as $v) add_post_meta($new_id, $k, maybe_unserialize($v));
            }
            $meta_key_key = "_{$k}";
            $vals2 = get_post_meta($src_id, $meta_key_key);
            if ($vals2) {
                delete_post_meta($new_id, $meta_key_key);
                foreach ($vals2 as $v) add_post_meta($new_id, $meta_key_key, maybe_unserialize($v));
            }
        }

        // Копируем специальные мета
        foreach ($special_meta as $m) {
            $vals = get_post_meta($src_id, $m);
            if ($vals) {
                delete_post_meta($new_id, $m);
                foreach ($vals as $v) add_post_meta($new_id, $m, maybe_unserialize($v));
            }
        }

        // Копируем остальные пользовательские мета
        $all_meta = get_post_meta($src_id);
        foreach ($all_meta as $meta_key => $values) {
            if (in_array($meta_key, array_merge($acf_keys, array_map(function($k){ return "_{$k}"; }, $acf_keys), $special_meta), true)) {
                continue;
            }
            $lower = strtolower($meta_key);
            if (strpos($lower, 'pll') !== false || strpos($lower, '_pll') !== false || strpos($lower, 'icl_') !== false || $meta_key === '_translations' || $meta_key === 'translations') {
                continue;
            }
            foreach ($values as $v) {
                add_post_meta($new_id, $meta_key, maybe_unserialize($v));
            }
        }

        // Устанавливаем язык и связываем переводы
        pll_set_post_language($new_id, 'ru');
        pll_save_post_translations(['uk' => $src_id, 'ru' => $new_id]);
        
        $cloned_count++;
        $log_messages[] = "Пост ID {$src_id} успешно клонирован в новый пост ID {$new_id}.";
    }

    echo "Готово. Создано {$cloned_count} новых постов.";
    echo "<h3>Подробности:</h3>";
    echo "<ul><li>" . implode("</li><li>", $log_messages) . "</li></ul>";
    exit;
});
```
---

# ✅ Памятка по выводу основного контента в WordPress

```php
// Вывод шорткода в шаблоне
<?=do_shortcode('111');?>

// Заголовок поста
<?php the_title(); ?>

// Контент поста
<?php the_content(); ?>

// Короткое описание (excerpt)
<?php the_excerpt(); ?>

// Ссылка на пост
<?php the_permalink(); ?>

// Дата публикации
<?php the_date(); ?>

// Автор поста
<?php the_author(); ?>

// ID поста
<?php the_ID(); ?>

// Категории поста
<?php the_category(', '); ?>

// Заголовок категории (в шаблоне категории)
<?php single_cat_title(); ?>

// Вывод acf поля в шаблоне категории
<?php
$term = get_queried_object();
$field = get_field( 'имя_поля', 'category_' . $term->term_id );
if ( $field ){echo esc_html( $field );}

// вывод репитера
$rows = get_field( 'cat_add_question', 'category_' . $term->term_id );
if ( $rows ) : ?>
    <?php foreach ( $rows as $row ) : ?>
          <?php echo esc_html( $row['question'] ); ?>
          <?php echo wp_kses_post( $row['answer'] ); ?>
	<?php endforeach; ?>
<?php endif; ?>
?>

//прямое обращение к массиву репитера

block_4_add_1
├──title
├──text
├──icon

//первая итерация
<?= get_field_object('block_4_add_1')['value'][0]['title']; ?>
<?= get_field_object('block_4_add_1')['value'][0]['title']; ?>
<?php $icon = get_field_object('block_4_add_1')['value'][0]['icon']; ?>
<img src="<?= esc_url($icon['url']); ?>" alt="<?= esc_attr($icon['alt']); ?>">



// Теги поста
<?php the_tags(); ?>

// Миниатюра поста (Featured Image)
<?php if ( has_post_thumbnail() ) {
    the_post_thumbnail('full'); // Размеры: 'thumbnail', 'medium', 'large', 'full'
} ?>

//acf изображение формат массив без класса
<?php if ( $img = get_field('photo_doctor_post')) echo wp_get_attachment_image($img['ID'], 'full'); ?>
//или с классом
<?php if($i=get_field('image_field')) echo '<img src="'.esc_url($i['url']).'" alt="'.esc_attr($i['alt']).'" class="111">'; ?>



// Миниатюра поста расширенные атрибуты
<?php
$id = get_post_thumbnail_id();
echo $id ? '<img src="' . esc_url( wp_get_attachment_image_url( $id, 'full' ) ) . '" class="" alt="' . esc_attr( get_post_meta( $id, '_wp_attachment_image_alt', true ) ) . '">' : '';
?>



// Цикл WordPress (The Loop)
<?php if ( have_posts() ) : while ( have_posts() ) : the_post(); ?>
    <h2><?php the_title(); ?></h2>
    <div><?php the_content(); ?></div>
<?php endwhile; endif; ?>

// Произвольные поля (Custom Fields)
<?php echo get_post_meta(get_the_ID(), 'meta_key', true); ?>

// ACF поля
<?php the_field('acf_field_name'); ?>

// Проверка наличия постов
<?php if ( have_posts() ) : ?>
    <!-- есть посты -->
<?php else : ?>
    <!-- постов нет -->
<?php endif; ?>

// Ссылки на предыдущий и следующий пост
<?php previous_post_link(); ?>
<?php next_post_link(); ?>

// WP_Query – получение списка постов
<?php
$args = array(
    'post_type' => 'post',
    'posts_per_page' => 5
);
$loop = new WP_Query($args);
while ($loop->have_posts()) : $loop->the_post();
    the_title();
    the_excerpt();
endwhile;
wp_reset_postdata();
?>

// Получение заголовка и ссылки страницы по ID
<?php echo get_the_title($post_id); ?>
<?php echo get_permalink($post_id); ?>

// Получение текущего URL поста
<?php echo get_permalink(); ?>

//ссылка на главную для polylang
<a href="<?= esc_url( function_exists('pll_home_url') ? pll_home_url() : home_url('/') ); ?>">Главная</a>
```


## ✅  Получить данные со страницы Общие настройки
```php
get_option( 'blogname' ); //название сайта

get_option( 'blogdescription' ); //подзаголовок (описание сайта)

get_option( 'admin_email' ); //email администратора

get_option( 'siteurl' ); //URL сайта

get_option( 'home' ); //адрес главной страницы
```
---

## ✅  Pretty url для Filter Everything free

### Добавляем в wp-config.php перед /* That's all, stop editing! Happy publishing. */:

```php
define('FLRT_PERMALINKS_ENABLED', true); //нужно для pretty urls Filter Everything
```
### Создаем плагин с кодом в папке /wp-content/plugins/filter-pretty-urls файл filter-pretty-urls.php:

```php
<?php
/**
 * Plugin Name: Filter Everything Pretty URLs (Free Hack)
 * Description: Делает красивые урлы для Filter Everything free (type и manuf)
 * Author: Roman Fix
 */

if ( ! defined('ABSPATH') ) exit;

/**
 * 1. Генерация красивого урла вместо query vars
 */
add_filter('flrt_get_filter_url', function($url, $filters, $args){
    if( empty($filters) ){
        return $url;
    }

    // Базовый URL (текущая категория)
    $base = strtok($url, '?');

    $segments = [];

    foreach( $filters as $filter ){
        $name   = $filter['taxonomy'];   // например type или manuf
        $values = implode('-', $filter['values']);
        $segments[] = $name . '-' . $values;
    }

    return trailingslashit( $base . '/' . implode('/', $segments) );
}, 20, 3);


/**
 * 2. Правила перезаписи: /.../type-aaa-bbb/manuf-ccc/
 */
add_action('init', function(){

    add_rewrite_rule(
        '^product-category/([^/]+(?:/[^/]+)*)/type-([^/]+)/manuf-([^/]+)/?$',
        'index.php?category_name=$matches[1]&type=$matches[2]&manuf=$matches[3]',
        'top'
    );

    add_rewrite_rule(
        '^product-category/([^/]+(?:/[^/]+)*)/type-([^/]+)/?$',
        'index.php?category_name=$matches[1]&type=$matches[2]',
        'top'
    );

    add_rewrite_rule(
        '^product-category/([^/]+(?:/[^/]+)*)/manuf-([^/]+)/?$',
        'index.php?category_name=$matches[1]&manuf=$matches[2]',
        'top'
    );
});

/**
 * 3. Регистрируем переменные, чтобы WP их понимал
 */
add_filter('query_vars', function($vars){
    $vars[] = 'type';
    $vars[] = 'manuf';
    return $vars;
});

```

---

## ✅  Глобальный массив поста
```php
global $post;
$post_data = get_post( get_the_ID(), ARRAY_A );

// добавим все метаполя
$post_data['meta'] = get_post_meta( get_the_ID() );

echo '<pre>';
print_r($post_data);
echo '</pre>';
```
---

## ✅  Функция для подсчета просмотров статей

### Добавляем в functions.php:
```php
// счетчик просмотра статей блога
// Функция для увеличения просмотров
function set_post_views($post_id) {
    $count = get_post_meta($post_id, 'post_views_count', true);
    $count = $count ? $count + 1 : 1; // Увеличиваем счетчик на 1
    update_post_meta($post_id, 'post_views_count', $count);
}

// Хук для вызова функции при загрузке поста
function track_post_views() {
    if (is_single()) {
        global $post;
        set_post_views($post->ID);
    }
}
add_action('wp_head', 'track_post_views');

// Функция для вывода количества просмотров
function get_post_views($post_id) {
    $count = get_post_meta($post_id, 'post_views_count', true);
    return $count ? $count : '0';
}

// Обнуляем просмотры для дублированной записи
function reset_post_views_on_duplicate($post_id, $post) {
    if ($post->post_type === 'post' && isset($_GET['action']) && $_GET['action'] === 'rd_duplicate_post_as_draft') {
        update_post_meta($post_id, 'post_views_count', 0);
    }
}
add_action('save_post', 'reset_post_views_on_duplicate', 10, 2);

// Обнуляем просмотры при создании перевода через Polylang
function reset_post_views_on_polylang_translation($post_id, $post) {
    // Проверяем, что это создание перевода
    if (function_exists('pll_get_post') && isset($_POST['pll_post_translations'])) {
        $translations = $_POST['pll_post_translations'];
        if (isset($translations['default']) && $translations['default'] != $post_id) {
            // Это перевод, обнуляем просмотры
            update_post_meta($post_id, 'post_views_count', 0);
        }
    }
}
add_action('save_post', 'reset_post_views_on_polylang_translation', 10, 2);


//счетчик просмотра статей блога
```
---

## ✅  Функция для подсчета времени чтения статьи

### Добавляем в functions.php:

```php
//Функция для подсчета времени чтения статьи
//стандартное поле контент
//<?=get_reading_time() . ' ' . pll__('time')? >
//acf поле
//<?=get_reading_time(get_the_ID(), 'desc_post_treatment') . ' ' . pll__('time')? >

function get_reading_time($post_id = null, $field_name = null) {
    if (!$post_id) {
        $post_id = get_the_ID();
    }

    // Если указано ACF-поле → берём его
    if ($field_name) {
        $content = get_field($field_name, $post_id);
    } else {
        // иначе стандартный контент
        $content = get_post_field('post_content', $post_id);
    }

    if (!$content) {
        return 1;
    }

    $content = strip_tags($content);

    // Подсчёт слов: кириллица + латиница + цифры
    preg_match_all('/[\p{L}\p{N}\']+/u', $content, $matches);
    $word_count = count($matches[0]);

    // Средняя скорость чтения
    $words_per_minute = 180;
    $minutes = ceil($word_count / $words_per_minute);

    return $minutes < 1 ? 1 : $minutes;
}
```
---

## ✅  Отзывы Google (посты и категории)

### Создаем группу полей "Отзывы Google" с полями:

1. Поле 1 - Файл "google_reviews_json" - Файл (массив-все)
2. Поле 2 - Выбор отзывов "selected_reviews" - выбор (select) - выбрать несколько - да
3. Назначаем в каких шаблонах выводить

### Код для functions.php

```php
// Отзывы Google

// ============================================================================
//   1. Подгружаем отзывы в select ACF (работает и в постах, и в категориях)
// ============================================================================
add_filter('acf/load_field/name=selected_reviews', function ($field) {

    // ★ NEW: определяем корректный ACF ID для терма в админке (category_123)
    $acf_id = false;

    // 1) Если мы в админке и явно редактируем термин (обычный WP-редактор терма)
    if (is_admin() && isset($_GET['tag_ID'])) {
        $term_id  = intval($_GET['tag_ID']);
        $taxonomy = isset($_GET['taxonomy']) ? sanitize_text_field($_GET['taxonomy']) : 'category';
        if ($term_id) {
            $acf_id = $taxonomy . '_' . $term_id; // формат ACF для термов: {taxonomy}_{term_id}
        }
    }

    // 2) Ещё один вариант получения терма (на случай нестандартных страниц)
    if (!$acf_id && is_admin()) {
        $queried = get_queried_object();
        if ($queried && isset($queried->taxonomy) && isset($queried->term_id)) {
            $acf_id = $queried->taxonomy . '_' . intval($queried->term_id);
        }
    }

    // 3) Фоллбэк — стандартный ACF helper (работает для постов и, в большинстве случаев, для термов)
    if (!$acf_id) {
        $acf_id = acf_get_valid_post_id(null);
    }

    // Safety: если всё ещё пусто — не ломаем админку, возвращаем поле без choices
    if (!$acf_id) {
        return $field;
    }

    // Получаем файл (работает и для post_123, и для category_45 и т.д.)
    $file = get_field('google_reviews_json', $acf_id);

    if ($file && isset($file['url'])) {
        $json = @file_get_contents($file['url']); // @ чтобы избежать warning, если URL недоступен
        $data = $json ? json_decode($json, true) : null;

        if (is_array($data) && isset($data['reviews'])) {
            $choices = [];

            foreach ($data['reviews'] as $review) {
                $id    = $review['id'] ?? '';
                $stars = isset($review['stars']) ? str_repeat('★', (int)$review['stars']) : '';
                $date  = $review['date'] ?? '';
                $text  = isset($review['text']) ? mb_strimwidth($review['text'], 0, 180, '...') : '';

                if ($id === '') continue;

                $label = trim("{$stars} · {$date} · {$review['authorName']} — {$text}");
                $choices[$id] = $label;
            }

            $field['choices'] = $choices;
        }
    }

    return $field;
});



// ============================================================================
//   2. Шорткод [selected_google_reviews] — теперь работает и в категориях
// ============================================================================
add_shortcode('selected_google_reviews', function () {

    // Определяем текущий объект корректно
    $obj = get_queried_object();

    if (!$obj) {
        return '';
    }

    // Определяем $acf_id вручную, без fallback
    if (!empty($obj->term_id)) {
        // Таксономия
        $acf_id = $obj->taxonomy . '_' . $obj->term_id;
    } else {
        // Пост
        $acf_id = $obj->ID ?? 0;
    }

    // Получаем поля только для этого ACF ID
    $json_file    = get_field('google_reviews_json', $acf_id);
    $selected_ids = get_field('selected_reviews', $acf_id);

    // Если админ не выбрал отзывы → ничего не выводим
    if (empty($selected_ids) || !is_array($selected_ids)) {
        return '';
    }

    // Нет JSON файла → не выводим
    if (!$json_file || empty($json_file['url'])) {
        return '';
    }

    // Загружаем JSON
    $json = @file_get_contents($json_file['url']);
    $data = json_decode($json, true);

    if (!is_array($data) || empty($data['reviews'])) {
        return '';
    }

    $reviews = $data['reviews'];

    // Индексируем по ID
    $indexed = [];
    foreach ($reviews as $r) {
        $indexed[$r['id']] = $r;
    }

    // Данные страницы
    $google_url   = $data['pageUrl']     ?? '';
    $ratingValue  = $data['ratingValue'] ?? 0;
    $reviewCount  = $data['reviewCount'] ?? 0;

    // Язык интерфейса
    $lang = function_exists('pll_current_language') ? pll_current_language() : 'uk';
    $title     = ($lang === 'uk') ? 'Відгуки про нас на Google' : 'Отзывы о нас на Google';
    $link_text = ($lang === 'uk') ? 'Подивитись усі відгуки' : 'Посмотреть все отзывы';

    // HTML
    $output  = '<div class="selected-google-reviews">';
    $output .= '<h3 class="google-reviews-title">' . esc_html($title) . '</h3>';
    $output .= '<div class="google-reviews">';

    $schema_reviews = [];

    foreach ($selected_ids as $id) {
        if (!isset($indexed[$id])) continue;

        $r = $indexed[$id];

        $avatar = esc_url($r['avatar']);
        $author = esc_html($r['authorName']);
        $stars  = (int)$r['stars'];
        $date   = esc_html($r['date']);
        $text   = esc_html($r['text']);

        // HTML
        $output .= '<div class="google-review">';
        $output .= '<p class="review_ava_fio"><img src="'.$avatar.'" alt="'.$author.'"> <strong>'.$author.'</strong></p>';
        $output .= '<p class="review_stars"><span>' . str_repeat("★", $stars) . '</span> · ' . $date . '</p>';
        $output .= '<p class="review_text">'.$text.'</p>';
        $output .= '</div>';

        // JSON-LD
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

    $output .= '</div>';

    if ($google_url) {
        $output .= '<div class="google_rev_link"><a href="' . esc_url($google_url) . '" target="_blank">' . esc_html($link_text) . '</a></div>';
    }

    $output .= '</div>';

    // JSON-LD
    if ($schema_reviews) {
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

        $output .= '<script type="application/ld+json">'
                 . wp_json_encode($organization, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES | JSON_PRETTY_PRINT)
                 . '</script>';
    }

    return $output;
});



// ============================================================================
//   3. Блок напоминания шорткода под ACF полем выбранных отзывов
// ============================================================================
add_action('acf/render_field/name=selected_reviews', function ($field) {

    // Показываем только при редактировании постов (не категорий)
    if ( ! is_admin() ) {
        return;
    }

    $screen = get_current_screen();

    // Условие: мы на странице редактирования поста, а не терминов
    if (
        ! $screen
        || $screen->base !== 'post'
        || ! empty($screen->taxonomy)  // если taxonomy не пуст — это терм
    ) {
        return;
    }
    ?>

    <div style="margin-top:10px; background:#f9f9f9; padding:10px; border-left:4px solid #0073aa;">
        <strong>📋 Шорткод для вставки:</strong><br>

        <div style="
            font-size:16px;
            background:#fff;
            border:1px solid #ddd;
            padding:4px 6px;
            display:inline-block;
            margin-top:4px;
            user-select:none;
        " contenteditable="false">
            <span style="user-select:all;">[selected_google_reviews]</span>
        </div>

        <p style="margin:5px 0 0; font-size:13px;">
            Выберите файл с отзывами, сохраните страницу, выберите отзывы и вставьте этот шорткод в контент и опять сохраните страницу.
        </p>
    </div>

    <?php
});


// ============================================================================
//   4. Разрешаем JSON для ACF поля "Файл"
// ============================================================================
add_filter('acf/fields/file/query', function ($args, $field, $post_id) {
    $args['post_mime_type'] = ['application/json'];
    return $args;
}, 10, 3);


// ============================================================================
//   5. Увеличиваем высоту select поля selected_reviews
// ============================================================================
function my_acf_admin_styles() {
    echo '<style>
        div[data-name="selected_reviews"] select,
        tr[data-name="selected_reviews"] select
         {
            height: 300px !important;
            max-width: 100% !important;
        }
    </style>';
}
add_action('admin_head', 'my_acf_admin_styles');

// Отзывы Google
```

---

```php
<!-- блок отзывы -->
<?=do_shortcode('[selected_google_reviews]');?>
<!-- блок отзывы -->
```

---

```css
/* блок гугл отзывов */

.selected-google-reviews{
    margin-top: 20px;
}
.google-reviews ul{
  display: inline;
}

.google-reviews-title
 {
    font-size: 24px;
    padding: 20px 0;
    color: #1b87ac;
    font-weight: 600;
    text-align: center;
}


.article-page .google-reviews-title{
    font-size: 20px;
    text-align: left;
    font-weight: 900;
    color: #333;
}

.selected-google-reviews h3 a{
    font-size: 20px;
    padding: 20px 0;
    color: #1b87ac;
    font-weight: 600;
    text-align: center;
}

.selected-google-reviews div a:hover{
  scale:1.05;
}


.google-reviews{
  padding: 0 40px;
}

.google-review{
    margin-bottom:20px;
    padding:5px;
}

.google-review p strong{
    font-size: 15px;
}

.google-review p img{
    margin: 0;
}


.google-reviews button.slick-next.slick-arrow {
    width: 40px;
    height: auto;
}

.google-reviews button.slick-prev.slick-arrow {
    width: 40px;
    height: auto;
}

.google-reviews .slick-next {
    right: 0;
}

.google-reviews .slick-prev {
    left: 0;
}

.google_rev_link{
    margin:0 0 40px 0;
    text-align:center;
}


.google_rev_link a{
    overflow: hidden;
    position: relative;
    padding: 10px 20px;
    background: -o-linear-gradient(357.17deg, #0ebc98 3.97%, #1c84ad 100%);
    background: linear-gradient(92.83deg, #0ebc98 3.97%, #1c84ad 100%);
    color: #fff;
    border-radius: 6px;
}

.google_rev_link a:hover{
    text-decoration: none;
    background: linear-gradient(92.83deg, #1c84ad 3.97%, #0ebc98 97.82%);
}

.google-reviews .review_ava_fio{
    display:flex;
    align-items:center;
}

.google-reviews .review_ava_fio img{
    width: 36px;
    height: 36px;
    border-radius:50%;
    margin-right:8px;
}

.google-reviews .review_stars span{
    color: #f39c12;
}


.google-reviews .review_text{
    font-size: 16px;
    margin-top:5px;
    max-height:200px;
    overflow-y:auto;
}

/* блок гугл отзывов */
```

---

```js
//инициируем слайдер гугл отзывов
$(document).ready(function() {
    $('.google-reviews').slick({
        slidesToShow: 3,
        slidesToScroll: 1,
        infinite: true,
        arrows: true,
        prevArrow: '<button type="button" class="slick-prev"><img src="/wp-content/themes/life/assets/images/prev.svg" alt="Previous"></button>',
        nextArrow: '<button type="button" class="slick-next"><img src="/wp-content/themes/life/assets/images/next.svg" alt="Next"></button>',
        dots: false,
        responsive: [{
            breakpoint: 1024,
            settings: {
                slidesToShow: 2
            }
        }, {
            breakpoint: 600,
            settings: {
                slidesToShow: 2
            }
        }, {
            breakpoint: 480,
            settings: {
                slidesToShow: 1,
                adaptiveHeight: true
            }
        }]
    });
});
//инициируем слайдер гугл отзывов
```

