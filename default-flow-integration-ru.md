# FLOW Внутренняя интеграция — Руководство по настройке

---

## 1. Скрипт скрытых полей

Добавляет все GET-параметры URL как скрытые поля формы при загрузке страницы, чтобы трекинговые параметры (fbclid, subid, pixel и др.) передавались в `order.php`.

**Размещение:** конец `index.html` / `index.php`, перед тегом `</body>`

```html
<script>
    const params = new URLSearchParams(window.location.search);
    const form = document.getElementById('order-form');
    params.forEach((value, key) => {
        const input = document.createElement('input');
        input.type = 'hidden';
        input.name = key;
        input.value = value;
        form.appendChild(input);
    });
</script>
```

**Требования:**
- Форма должна иметь `id="order-form"` — убедитесь, что он совпадает с вызовом `getElementById` выше
- Форма должна указывать на скрипт обработки заказа:

```html
<form id="order-form" action="order.php" method="POST">
```

---

## 2. Интеграция Meta Pixel

### 2.1 Лендинг (`index.html` / `index.php`)

Считывает `pixel` из URL и инициализирует FB Pixel с этим значением.

**Размещение — перед `</head>`:**

```html
<!-- Meta Pixel Code -->
<script>
    const urlParams = new URLSearchParams(window.location.search);
    const pixelId = urlParams.get('pixel');
    !(function (f, b, e, v, n, t, s) {
        if (f.fbq) return;
        n = f.fbq = function () {
            n.callMethod ? n.callMethod.apply(n, arguments) : n.queue.push(arguments);
        };
        if (!f._fbq) f._fbq = n;
        n.push = n; n.loaded = !0; n.version = '2.0'; n.queue = [];
        t = b.createElement(e); t.async = !0; t.src = v;
        s = b.getElementsByTagName(e)[0];
        s.parentNode.insertBefore(t, s);
    })(window, document, 'script', 'https://connect.facebook.net/en_US/fbevents.js');
    fbq('init', pixelId);
    fbq('track', 'PageView');
</script>
<!-- End Meta Pixel Code -->
```

**Размещение — перед `</body>`:**

```html
<!-- Meta Pixel noscript -->
<noscript id="fb-noscript"></noscript>
```

---

### 2.2 Страница благодарности (`ok.php` / `ok.html`)

Та же инициализация пикселя, но дополнительно фиксирует событие конверсии `Lead`.

**Размещение — перед `</head>`:**

```html
<!-- Meta Pixel Code -->
<script>
    const urlParams = new URLSearchParams(window.location.search);
    const pixelId = urlParams.get('pixel');
    !(function (f, b, e, v, n, t, s) {
        if (f.fbq) return;
        n = f.fbq = function () {
            n.callMethod ? n.callMethod.apply(n, arguments) : n.queue.push(arguments);
        };
        if (!f._fbq) f._fbq = n;
        n.push = n; n.loaded = !0; n.version = '2.0'; n.queue = [];
        t = b.createElement(e); t.async = !0; t.src = v;
        s = b.getElementsByTagName(e)[0];
        s.parentNode.insertBefore(t, s);
    })(window, document, 'script', 'https://connect.facebook.net/en_US/fbevents.js');
    fbq('init', pixelId);
    fbq('track', 'PageView');
    fbq('track', 'Lead');
</script>
<!-- End Meta Pixel Code -->
```

**Размещение — перед `</body>`:**

```html
<!-- Meta Pixel noscript -->
<noscript id="fb-noscript"></noscript>
```

**Проверка пикселя:** добавьте `?pixel=1230989095054222` к URL, отключите AdBlock и проверьте результат через расширение Chrome **Meta Pixel Helper**.

---

## 3. Интеграция `order.php`

Создайте файл `order.php` в корне лендинга. Заполните три конфигурационных значения (уточнить у менеджера).

```php
<?php
$url     = 'https://t.trklinkx.com/click';
$apiKey  = '';    // уточнить у менеджера
$pid     = 1;     // уточнить у менеджера
$offerId = 1;     // уточнить у менеджера
$country = 'RO';

$data = [
    "sub30"       => $apiKey,
    "pid"         => $pid,
    "offer_id"    => $offerId,
    "sub9"        => $_POST["name"]         ?? "",
    "sub12"       => $_POST["phone"]        ?? "",
    "cpa_offer_id"=> $_POST["cpa_offer_id"] ?? "",
    "fbclid"      => $_POST["fbclid"]       ?? "",
    "subid"       => $_POST["subid"]        ?? "",
    "sub1"        => $_POST["sub1"]         ?? "",
    "sub2"        => $_POST["sub2"]         ?? "",
    "sub3"        => $_POST["sub3"]         ?? "",
    "pixel"       => $_POST["pixel"]        ?? "",
    "country"     => $country,
    "ua"          => $_SERVER["HTTP_USER_AGENT"],
    "ip"          => $_SERVER["HTTP_CF_CONNECTING_IP"]
                     ?: ($_SERVER["HTTP_X_FORWARDED_FOR"]
                     ?: $_SERVER["REMOTE_ADDR"]),
];

$curl = curl_init($url . '?' . http_build_query($data));
curl_setopt_array($curl, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_FOLLOWLOCATION => true,
    CURLOPT_TIMEOUT        => 5,
    CURLOPT_SSL_VERIFYPEER => 0,
]);

$response = curl_exec($curl);
curl_close($curl);

$pixel = $_POST["pixel"] ?? "";
header("Location: ok.php?pixel=$pixel");
exit();
```

**Редирект в конце скрипта должен указывать на страницу благодарности** (`ok.php` по умолчанию). Обновите имя файла, если страница благодарности называется иначе:

```php
header("Location: ok.php?pixel=$pixel");
```

**Конфигурационная таблица:**

| Переменная | Откуда взять       |
|------------|--------------------|
| `$apiKey`  | Уточнить у менеджера |
| `$pid`     | Уточнить у менеджера |
| `$offerId` | Уточнить у менеджера |
| `$country` | Установить по офферу |

---

## Чеклист интеграции

- [ ] `<form id="order-form" action="order.php" method="POST">` — правильный id и action
- [ ] Скрипт скрытых полей добавлен перед `</body>` на странице лендинга
- [ ] Meta Pixel `<script>` добавлен перед `</head>` на странице лендинга
- [ ] Meta Pixel `<noscript>` добавлен перед `</body>` на странице лендинга
- [ ] Meta Pixel `<script>` (с событием `Lead`) добавлен перед `</head>` на странице благодарности
- [ ] Meta Pixel `<noscript>` добавлен перед `</body>` на странице благодарности
- [ ] `order.php` создан с правильными `$apiKey`, `$pid`, `$offerId`, `$country`
- [ ] Редирект в `order.php` указывает на правильное имя файла страницы благодарности
- [ ] Пиксель проверен через расширение Meta Pixel Helper
