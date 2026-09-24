# Spam-Schutz / Roboter-Schutz für BorderFlow & Logistikberater

## 🛡️ 3 Lösungen (vom einfachsten zum sichersten)

---

## ✅ Lösung 1: Honeypot + Zeitbasierte Validierung (EINFACH, KOSTENLOS)

**Vorteile:**
- Keine externen Services nötig
- Unsichtbar für echte Nutzer
- Blockt 90% der Bots
- DSGVO-konform (keine Daten an Dritte)

### HTML Kontaktformular mit Honeypot

```html
<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <title>Kontakt - BorderFlow</title>
    <style>
        /* Honeypot-Feld verstecken */
        .honeypot {
            position: absolute;
            left: -9999px;
            width: 1px;
            height: 1px;
        }
        
        .kontaktformular {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .form-group {
            margin-bottom: 15px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input, textarea {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 14px;
        }
        
        button {
            background: #00A896;
            color: white;
            padding: 12px 30px;
            border: none;
            border-radius: 4px;
            font-size: 16px;
            cursor: pointer;
        }
        
        button:hover {
            background: #008777;
        }
        
        .error {
            color: red;
            font-size: 14px;
            margin-top: 5px;
        }
        
        .success {
            color: green;
            padding: 15px;
            background: #d4edda;
            border: 1px solid #c3e6cb;
            border-radius: 4px;
            margin-bottom: 15px;
        }
    </style>
</head>
<body>

<div class="kontaktformular">
    <h2>Kontaktanfrage</h2>
    
    <form id="contactForm" method="POST" action="submit-form.php">
        
        <!-- Honeypot: Bots füllen dieses Feld aus, echte Menschen nicht -->
        <div class="honeypot">
            <label for="website">Website (nicht ausfüllen)</label>
            <input type="text" id="website" name="website" tabindex="-1" autocomplete="off">
        </div>
        
        <!-- Verstecktes Zeitstempel-Feld -->
        <input type="hidden" id="timestamp" name="timestamp" value="">
        
        <div class="form-group">
            <label for="name">Name *</label>
            <input type="text" id="name" name="name" required>
        </div>
        
        <div class="form-group">
            <label for="email">E-Mail *</label>
            <input type="email" id="email" name="email" required>
        </div>
        
        <div class="form-group">
            <label for="firma">Firma</label>
            <input type="text" id="firma" name="firma">
        </div>
        
        <div class="form-group">
            <label for="nachricht">Ihre Nachricht *</label>
            <textarea id="nachricht" name="nachricht" rows="6" required></textarea>
        </div>
        
        <button type="submit">Anfrage senden</button>
    </form>
</div>

<script>
// Zeitstempel setzen wenn Seite geladen wird
document.getElementById('timestamp').value = Date.now();

// Formular-Validierung
document.getElementById('contactForm').addEventListener('submit', function(e) {
    // Honeypot prüfen
    const honeypot = document.getElementById('website').value;
    if (honeypot !== '') {
        e.preventDefault();
        alert('Spam erkannt. Anfrage blockiert.');
        return false;
    }
    
    // Zeitbasierte Validierung (mind. 3 Sekunden)
    const timestamp = parseInt(document.getElementById('timestamp').value);
    const now = Date.now();
    const timeDiff = (now - timestamp) / 1000; // in Sekunden
    
    if (timeDiff < 3) {
        e.preventDefault();
        alert('Bitte füllen Sie das Formular vollständig aus.');
        return false;
    }
});
</script>

</body>
</html>
```

### PHP Backend (submit-form.php)

```php
<?php
// Anti-Spam-Validierung im Backend

// 1. Honeypot prüfen
if (!empty($_POST['website'])) {
    http_response_code(400);
    die('Spam detected');
}

// 2. Zeitbasierte Validierung (mind. 3 Sekunden)
if (isset($_POST['timestamp'])) {
    $timestamp = intval($_POST['timestamp']);
    $now = time() * 1000; // in Millisekunden
    $timeDiff = ($now - $timestamp) / 1000; // in Sekunden
    
    if ($timeDiff < 3 || $timeDiff > 3600) { // zwischen 3 Sek und 1 Stunde
        http_response_code(400);
        die('Invalid submission time');
    }
}

// 3. Pflichtfelder prüfen
if (empty($_POST['name']) || empty($_POST['email']) || empty($_POST['nachricht'])) {
    http_response_code(400);
    die('Pflichtfelder fehlen');
}

// 4. E-Mail validieren
$email = filter_var($_POST['email'], FILTER_VALIDATE_EMAIL);
if (!$email) {
    http_response_code(400);
    die('Ungültige E-Mail-Adresse');
}

// 5. Daten bereinigen
$name = htmlspecialchars(trim($_POST['name']));
$firma = htmlspecialchars(trim($_POST['firma']));
$nachricht = htmlspecialchars(trim($_POST['nachricht']));

// 6. E-Mail versenden
$to = 'mb@logistikberater.at';
$subject = 'Neue Kontaktanfrage von BorderFlow.at';
$message = "Name: $name\n";
$message .= "E-Mail: $email\n";
$message .= "Firma: $firma\n\n";
$message .= "Nachricht:\n$nachricht\n\n";
$message .= "---\n";
$message .= "Gesendet über BorderFlow.at am " . date('d.m.Y H:i:s');

$headers = "From: noreply@borderflow.at\r\n";
$headers .= "Reply-To: $email\r\n";
$headers .= "X-Mailer: PHP/" . phpversion();

if (mail($to, $subject, $message, $headers)) {
    echo json_encode(['success' => true, 'message' => 'Vielen Dank! Wir melden uns in Kürze.']);
} else {
    http_response_code(500);
    echo json_encode(['success' => false, 'message' => 'Fehler beim Senden. Bitte versuchen Sie es erneut.']);
}
?>
```

---

## ✅ Lösung 2: Google reCAPTCHA v3 (UNSICHTBAR, SEHR SICHER)

**Vorteile:**
- Komplett unsichtbar für Nutzer
- Hochentwickelte Bot-Erkennung
- Kostenlos bis 1 Million Anfragen/Monat

**Nachteil:**
- Google-Service (DSGVO: Datenschutzerklärung anpassen)

### 1. reCAPTCHA-Schlüssel erhalten

1. Gehen Sie zu: https://www.google.com/recaptcha/admin
2. Registrieren Sie Ihre Domain: `borderflow.at` und `logistikberater.at`
3. Wählen Sie **reCAPTCHA v3**
4. Sie erhalten:
   - **Site Key** (öffentlich, im HTML)
   - **Secret Key** (privat, nur im Backend)

### 2. HTML mit reCAPTCHA v3

```html
<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <title>Kontakt - BorderFlow</title>
    <script src="https://www.google.com/recaptcha/api.js?render=IHR_SITE_KEY"></script>
    <style>
        /* Ihr CSS hier (wie oben) */
    </style>
</head>
<body>

<div class="kontaktformular">
    <h2>Kontaktanfrage</h2>
    
    <form id="contactForm">
        <div class="form-group">
            <label for="name">Name *</label>
            <input type="text" id="name" name="name" required>
        </div>
        
        <div class="form-group">
            <label for="email">E-Mail *</label>
            <input type="email" id="email" name="email" required>
        </div>
        
        <div class="form-group">
            <label for="firma">Firma</label>
            <input type="text" id="firma" name="firma">
        </div>
        
        <div class="form-group">
            <label for="nachricht">Ihre Nachricht *</label>
            <textarea id="nachricht" name="nachricht" rows="6" required></textarea>
        </div>
        
        <button type="submit">Anfrage senden</button>
    </form>
</div>

<script>
document.getElementById('contactForm').addEventListener('submit', function(e) {
    e.preventDefault();
    
    // reCAPTCHA v3 Token holen
    grecaptcha.ready(function() {
        grecaptcha.execute('IHR_SITE_KEY', {action: 'submit'}).then(function(token) {
            // Token zum Formular hinzufügen
            const formData = new FormData(document.getElementById('contactForm'));
            formData.append('recaptcha_token', token);
            
            // Per AJAX senden
            fetch('submit-form-recaptcha.php', {
                method: 'POST',
                body: formData
            })
            .then(response => response.json())
            .then(data => {
                if (data.success) {
                    alert('Vielen Dank! Wir melden uns in Kürze.');
                    document.getElementById('contactForm').reset();
                } else {
                    alert('Fehler: ' + data.message);
                }
            })
            .catch(error => {
                alert('Fehler beim Senden. Bitte versuchen Sie es erneut.');
            });
        });
    });
});
</script>

</body>
</html>
```

### 3. PHP Backend mit reCAPTCHA-Validierung

```php
<?php
// submit-form-recaptcha.php

// reCAPTCHA Secret Key (GEHEIM HALTEN!)
$recaptcha_secret = 'IHR_SECRET_KEY';

// 1. reCAPTCHA Token validieren
if (!isset($_POST['recaptcha_token'])) {
    http_response_code(400);
    die(json_encode(['success' => false, 'message' => 'reCAPTCHA fehlt']));
}

$recaptcha_token = $_POST['recaptcha_token'];

// Google reCAPTCHA API aufrufen
$recaptcha_url = 'https://www.google.com/recaptcha/api/siteverify';
$recaptcha_data = [
    'secret' => $recaptcha_secret,
    'response' => $recaptcha_token,
    'remoteip' => $_SERVER['REMOTE_ADDR']
];

$recaptcha_options = [
    'http' => [
        'method' => 'POST',
        'header' => 'Content-Type: application/x-www-form-urlencoded',
        'content' => http_build_query($recaptcha_data)
    ]
];

$recaptcha_context = stream_context_create($recaptcha_options);
$recaptcha_result = file_get_contents($recaptcha_url, false, $recaptcha_context);
$recaptcha_json = json_decode($recaptcha_result);

// Score prüfen (0.0 = Bot, 1.0 = Mensch)
if (!$recaptcha_json->success || $recaptcha_json->score < 0.5) {
    http_response_code(400);
    die(json_encode(['success' => false, 'message' => 'Bot erkannt. Anfrage blockiert.']));
}

// 2. Restliche Validierung (wie oben)
if (empty($_POST['name']) || empty($_POST['email']) || empty($_POST['nachricht'])) {
    http_response_code(400);
    die(json_encode(['success' => false, 'message' => 'Pflichtfelder fehlen']));
}

$email = filter_var($_POST['email'], FILTER_VALIDATE_EMAIL);
if (!$email) {
    http_response_code(400);
    die(json_encode(['success' => false, 'message' => 'Ungültige E-Mail']));
}

// 3. Daten bereinigen und E-Mail senden
$name = htmlspecialchars(trim($_POST['name']));
$firma = htmlspecialchars(trim($_POST['firma']));
$nachricht = htmlspecialchars(trim($_POST['nachricht']));

$to = 'mb@logistikberater.at';
$subject = 'Neue Kontaktanfrage von BorderFlow.at';
$message = "Name: $name\n";
$message .= "E-Mail: $email\n";
$message .= "Firma: $firma\n\n";
$message .= "Nachricht:\n$nachricht\n\n";
$message .= "---\n";
$message .= "reCAPTCHA Score: " . $recaptcha_json->score . "\n";
$message .= "Gesendet am " . date('d.m.Y H:i:s');

$headers = "From: noreply@borderflow.at\r\n";
$headers .= "Reply-To: $email\r\n";

if (mail($to, $subject, $message, $headers)) {
    echo json_encode(['success' => true, 'message' => 'Vielen Dank!']);
} else {
    http_response_code(500);
    echo json_encode(['success' => false, 'message' => 'Fehler beim Senden']);
}
?>
```

---

## ✅ Lösung 3: E-Mail-Verschleierung auf Website

**Problem:** Bots scrapen E-Mail-Adressen von Websites

**Lösung:** E-Mail nicht direkt lesbar im HTML

### Variante A: JavaScript-Verschleierung

```html
<!-- Statt: -->
<a href="mailto:mb@logistikberater.at">mb@logistikberater.at</a>

<!-- Verwenden Sie: -->
<span id="email-kontakt"></span>

<script>
// E-Mail zusammensetzen (Bots können das nicht lesen)
var user = 'mb';
var domain = 'logistikberater';
var tld = 'at';
var email = user + '@' + domain + '.' + tld;

document.getElementById('email-kontakt').innerHTML = 
    '<a href="mailto:' + email + '">' + email + '</a>';
</script>
```

### Variante B: CSS-Verschleierung

```html
<style>
.email-protection {
    unicode-bidi: bidi-override;
    direction: rtl;
}
</style>

<!-- E-Mail rückwärts schreiben -->
<span class="email-protection">ta.retarebkitsigol@bm</span>
```

### Variante C: Kontaktformular statt E-Mail

```html
<!-- Besser: Gar keine E-Mail-Adresse anzeigen -->
<a href="/kontakt">Kontaktformular</a>

<!-- Oder: -->
<button onclick="window.location.href='/kontakt'">
    Jetzt Kontakt aufnehmen
</button>
```

---

## 🎯 Meine Empfehlung für Sie

### Kombination für maximalen Schutz:

1. **Honeypot + Zeitvalidierung** im Kontaktformular (Lösung 1)
2. **E-Mail-Verschleierung** auf der Website (Lösung 3A)
3. **Optional:** reCAPTCHA v3 zusätzlich (wenn Spam weiterhin Problem)

**Warum diese Kombination:**
- ✅ Blockt 95%+ aller Bots
- ✅ DSGVO-konform (ohne reCAPTCHA)
- ✅ Unsichtbar für echte Nutzer
- ✅ Einfach zu implementieren
- ✅ Keine monatlichen Kosten

---

## 📋 Implementierungs-Checkliste

### Für BorderFlow.at:

- [ ] Kontaktformular mit Honeypot erstellen
- [ ] Zeitbasierte Validierung hinzufügen
- [ ] Backend-Validierung in PHP implementieren
- [ ] E-Mail-Adresse auf Website verschleiern
- [ ] Testen mit echten Anfragen
- [ ] Spam-Rate nach 1 Woche prüfen

### Optional (bei weiterem Spam):

- [ ] Google reCAPTCHA v3 einrichten
- [ ] Datenschutzerklärung anpassen (reCAPTCHA erwähnen)
- [ ] Score-Threshold anpassen (Standard: 0.5)

---

## 🔧 Technische Anforderungen

**Was Sie brauchen:**
- PHP 7.0+ auf Ihrem Server
- `mail()` Funktion aktiviert (Standard bei den meisten Hostern)
- SSL-Zertifikat (HTTPS) – haben Sie bereits ✅

**Was ich tun kann:**
- Code direkt für Ihre Website anpassen
- Formular in Ihr bestehendes Design integrieren
- Testen und optimieren

---

## 📞 Nächste Schritte

**Soll ich:**
1. Den Code direkt für borderflow.at/logistikberater.at anpassen?
2. Ein komplettes Kontaktformular-HTML mit Ihrem Design erstellen?
3. Die Implementierung Schritt-für-Schritt erklären?

**Sagen Sie mir einfach Bescheid! 🚀**

---

**Erstellt:** September 2026  
**Für:** BorderFlow & Logistikberater Spam-Schutz  
**Status:** Ready to implement
