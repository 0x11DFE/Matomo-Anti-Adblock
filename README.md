# Matomo-Anti-Adblock
A way to bypass adblock for Matomo/Piwik

##### Note
This has only been tested agains't uBlock Origin
I take no credit for this, all info can be found [here](https://github.com/matomo-org/matomo/issues/7364)

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