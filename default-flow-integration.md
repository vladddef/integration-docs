# FLOW Internal Integration — Setup Guide

---

## 1. Hidden Inputs Script

Appends all URL query parameters as hidden form fields on page load, so tracking params (fbclid, subid, pixel, etc.) are forwarded to `order.php`.

**Placement:** end of `index.html` / `index.php`, just before `</body>`

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

**Requirements:**
- Form must have `id="order-form"` — verify it matches the `getElementById` call above
- Form must point to the order script:

```html
<form id="order-form" action="order.php" method="POST">
```

---

## 2. Meta Pixel Integration

### 2.1 Landing page (`index.html` / `index.php`)

Reads `pixel` from the URL and initializes the FB pixel with it.

**Placement — before `</head>`:**

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

**Placement — before `</body>`:**

```html
<!-- Meta Pixel noscript -->
<noscript id="fb-noscript"></noscript>
```

---

### 2.2 Thank You page (`ok.php` / `ok.html`)

Same pixel init as above but also fires the `Lead` conversion event.

**Placement — before `</head>`:**

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

**Placement — before `</body>`:**

```html
<!-- Meta Pixel noscript -->
<noscript id="fb-noscript"></noscript>
```

**Verify pixel works:** append `?pixel=1230989095054222` to the URL, disable AdBlock, and check with the **Meta Pixel Helper** Chrome extension.

---

## 3. `order.php` Integration

Create `order.php` in the landing root. Fill in the three config values (ask the manager).

```php
<?php
$url     = 'https://t.trklinkx.com/click';
$apiKey  = '';    // ask manager
$pid     = 1;     // ask manager
$offerId = 1;     // ask manager
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

**The redirect at the end of the script must point to the thank you page** (`ok.php` by default). Update the filename if the TYP page has a different name:

```php
header("Location: ok.php?pixel=$pixel");
```

**Config checklist:**

| Variable  | Where to get it |
|-----------|----------------|
| `$apiKey` | Ask manager     |
| `$pid`    | Ask manager     |
| `$offerId`| Ask manager     |
| `$country`| Set per offer   |

---

## Integration Checklist

- [ ] `<form id="order-form" action="order.php" method="POST">` — correct id and action
- [ ] Hidden inputs script added before `</body>` on index page
- [ ] Meta Pixel `<script>` added before `</head>` on index page
- [ ] Meta Pixel `<noscript>` added before `</body>` on index page
- [ ] Meta Pixel `<script>` (with `Lead` event) added before `</head>` on thank you page
- [ ] Meta Pixel `<noscript>` added before `</body>` on thank you page
- [ ] `order.php` created with correct `$apiKey`, `$pid`, `$offerId`, `$country`
- [ ] Redirect in `order.php` points to the correct TYP page filename
- [ ] Pixel verified with Meta Pixel Helper Chrome extension
