# Authenticated Stored XSS on Custom text for the floating widget field - Translate WordPress – Google Language Translator

## About the Plugin:
Version: 6.0.11  
Author: Translate AI Multilingual Solutions  
Last Updated: 2 weeks ago  
Requires WordPress Version: 2.9 or higher  
Compatible up to: 5.8  
Active Installations: 100,000+  
URL: https://wordpress.org/plugins/google-language-translator/  

## Vulnerability Detail: 
## Description: 
The vulnerability appears on the Custom text for the floating widget field, the parameter isn't escaped the HTML Injection. When taking a look at the source code, the parameter __floating_widget_text__ not covered by any filter and makes it's vulnerable to XSS, since the parameter is saving the value. The Stored XSS can be happening and stored on the __floating_widget_text__ parameter. 

## Vulnerable Code: 
File: google-languange-translator.php  
Line : 369 - 373  
```
    if( $is_active == 1) {
      if ($floating_widget=='yes') {
        $str.='<div id="glt-translate-trigger"><span'.($floating_widget_text_translation_allowed != 1 ? ' class="notranslate"' : ' class="translate"').'>'.(empty($floating_widget_text) ? 'Translate &raquo;' : $floating_widget_text).'</span></div>';
        $str.='<div id="glt-toolbar"></div>';
      } //endif $floating_widget
```

## Step To Reproduce: 
1. Go to Setting Translate WordPress – Google Language Translator
2. Input the HTML Payload / XSS payload such as `"<h1>asd</h1><img src=x onerror=alert(1)>"`
3. Save
4. The XSS will be executed

## Impact:
Stored XSS on the __floating_widget_text__ parameter and can be affected to various user page. 

## Fix:
Filter the __floating_widget_text__ parameter with __esc_attr__ function. 

## Fix Code, and already tested :

```
File: google-languange-translator.php
Line: 361

$floating_widget_text = esc_attr(get_option ('googlelanguagetranslator_floating_widget_text'));
```

## Timeline
05 August 2021 - Report To Vendor

Created this page is on 05 - August 2021