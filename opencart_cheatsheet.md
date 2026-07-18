# 🌟 Opencart Cheat Sheet


## ✅ Смотрим блок в режиме разработчика

```php
<!-- site.ua/ua/test?dev=1 -->

<?php if (isset($_GET['dev']) && $_GET['dev'] == 1) { ?>
    <div data-dev="dev"></div>
<?php } ?>
```

---

## ✅ вывод массива в лог файл из любого php файла

```php
file_put_contents(DIR_LOGS . 'error.log', print_r($res, true), FILE_APPEND);
```

---

