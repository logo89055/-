<?php
// Путь к исходному файлу
$source_file = 'image.png';

// Размеры нового изображения  200 на 100 пикселей
$new_width = 200;
$new_height = 100;

// Путь к кэшированному файлу
$cache_file = 'cache/image_' . $new_width . 'x' . $new_height . '.png';

// Если файл в кэше существует, возвращаем его
if (file_exists($cache_file)) {
    cacheControl(3600); // Установка кэширования на 1 час
    readfile($cache_file);
    exit;
}

// Загрузка исходного файла
$source_image = imagecreatefrompng($source_file);

// Создание нового изображения
$new_image = imagecreatetruecolor($new_width, $new_height);

// Масштабирование и центрирование
$source_width = imagesx($source_image);
$source_height = imagesy($source_image);
$aspect_ratio = $source_width / $source_height;
$new_aspect_ratio = $new_width / $new_height;

if ($new_aspect_ratio > $aspect_ratio) {
    $scaled_width = $new_width;
    $scaled_height = $new_width / $aspect_ratio;
    $source_x = 0;
    $source_y = ($source_height - $scaled_height) / 2;
} else {
    $scaled_width = $new_height * $aspect_ratio;
    $scaled_height = $new_height;
    $source_x = ($source_width - $scaled_width) / 2;
    $source_y = 0;
}

imagecopyresampled($new_image, $source_image, 0, 0, $source_x, $source_y, $new_width, $new_height, $scaled_width, $scaled_height);

// Оптимизация изображения
imagepng($new_image, $cache_file);

// Отправка нового изображения в браузер
cacheControl(3600); // Установка кэширования на 1 час
header('Content-Type: image/png');
imagepng($new_image);
imagedestroy($source_image);
imagedestroy($new_image);

function cacheControl($seconds)
{
    header('Cache-Control: max-age=' . $seconds . ', public');
    header('Expires: ' . gmdate('D, d M Y H:i:s', time() + $seconds) . ' GMT');
}
