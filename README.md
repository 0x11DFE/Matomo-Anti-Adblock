# Matomo-Anti-Adblock
A way to bypass adblock for Matomo/Piwik

#### Note
This has only been tested agains't uBlock Origin.

[More details about Matomo/Piwik adblock bypass](https://github.com/matomo-org/matomo/issues/7364)

## The "dynamic" way

#### Downside
- Header are set by the php file in order to replicate and return a valid javascript text
So it is unrealiable and not dependent on the server.

#### Why would you want to use this way?
- With this way after updating Matomo/Piwik you will not be forced to edit anything by hand.

### Procedee
 - Create a file in your Mamoto root directory e.x 1337.php and paste the following code
```php
if (empty($_GET)) {
    $modifiedSince = FALSE;
    if (isset($_SERVER['HTTP_IF_MODIFIED_SINCE'])) {
        $modifiedSince = $_SERVER['HTTP_IF_MODIFIED_SINCE'];
        // strip any trailing data appended to header
        if (FALSE !== ($semicolon = strpos($modifiedSince, ';'))) {
            $modifiedSince = strtotime(substr($modifiedSince, 0, $semicolon));
        }
    }
    // Re-Download the piwik.js once a week maximum
    $lastModified = time() - 604800;

    // set HTTP response headers
    header('Vary: Accept-Encoding');

    // Returns 304 if not modified since
    if (!empty($modifiedSince) && $modifiedSince <= $lastModified) {
        header($_SERVER['SERVER_PROTOCOL'] . ' 304 Not Modified');
    } else {
        header('Last-Modified: ' . gmdate('D, d M Y H:i:s') . ' GMT');
        @header('Content-Type: application/javascript; charset=UTF-8');
        if ($piwikJs = file_get_contents('piwik.js')) {
            $piwikJs = str_replace(['"action_name="', '"&idsite="'], ['"wannabe="', '"&1337="'], $piwikJs);
            echo $piwikJs;
        } else {
            header($_SERVER['SERVER_PROTOCOL'] . '505 Internal server error');
        }
    }
    exit;
}
```
 - Place on top of piwik.php the following code
```php
if (isset($_GET['wannabe'])) {
    $_GET['action_name'] = $_GET['wannabe'];
    unset($_GET['wannabe']);
}
if (isset($_GET['1337'])) {
    $_GET['idsite'] = $_GET['1337'];
    unset($_GET['1337']);
}
```

 - Then generate a tracking code and replace it to match the following code
 ```html
<script type="text/javascript">
var _paq = window._paq || [];
_paq.push(['trackPageView']);
_paq.push(['enableLinkTracking']);
(function () {
    var u = "https://example.com/";
    _paq.push(['setTrackerUrl', u + 'matomo.php']);
    _paq.push(['setSiteId', '1']);
    var d = document, g = d.createElement('script'), s = d.getElementsByTagName('script')[0];
            g.type = 'text/javascript';
            g.async = true;
            g.defer = true;
            g.src = u + '1337.php';
            s.parentNode.insertBefore(g, s);
        })();
</script>
<noscript><p><img src="https://example.com/matomo.php?1337=1&amp;rec=1" style="border:0;" alt=""/></p></noscript>
```


## The "static" way

#### Downside
- Everytime that Matomo/Piwik gets updated you will need to recreate/copy "matomo.js" to "1337.js"

#### Why would you want to use this way?
- The script called will be static and not modified and returned by a php file so the server will be able to set his header without issues.

### Procedee
- Copy piwik.js to 1337.js
- Replace manually "action_name" to "wannabe" and "idsite" to "1337" within the copied file "1337.js".
- Create a file called 1337.php
- Paste the php code bellow inside "1337.php"
- Generate your tracking code true Matomo panel and replace "matomo.php" to "1337.php" and "matomo.js" to "1337.js"

```php
<?php

// define piwik root path
if (!defined('PIWIK_DOCUMENT_ROOT')) {
    define('PIWIK_DOCUMENT_ROOT', dirname(__FILE__) == '/' ? '' : dirname(__FILE__));
}

// replace "action_name" by "wannabe"
if (isset($_GET['wannabe'])) {
    $_GET['action_name'] = $_GET['wannabe'];
    unset($_GET['wannabe']);
}

// replace "idsite" by "1337"
if (isset($_GET['1337'])) {
    $_GET['idsite'] = $_GET['1337'];
    unset($_GET['1337']);
}

include PIWIK_DOCUMENT_ROOT . '/piwik.php';
```
