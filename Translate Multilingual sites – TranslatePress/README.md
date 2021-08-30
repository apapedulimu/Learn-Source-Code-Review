# WordPress Plugin Translate Multilingual sites – TranslatePress version 2.0.6 - 'translated' Stored Cross-Site Scripting (XSS) (Authenticated)

## About The Finding
Date: 06-08-2021
Exploit Author: Nosa Shandy (Apapedulimu)
Vendor Homepage: https://translatepress.com/
Software Link: https://wordpress.org/plugins/translatepress-multilingual/
CVE : CVE-2021-24610
WPScan : https://wpscan.com/vulnerability/b87fcc2f-c2eb-4e23-9757-d1c590f26d3f

## About the Plugin:
Version: 2.0.6
Author: Cozmoslabs, Razvan Mocanu, Madalin Ungureanu, Cristophor Hurduban
Requires WordPress Version: 3.1.0 or higher
Compatible up to: 5.8
Requires PHP Version: 5.6.20 or higher
Active Installations: 100,000+ 
URL: https://wordpress.org/plugins/translatepress-multilingual/

## Vulnerability Detail: 
## Description: 
The plugin does not implement a proper filter on the 'translated' parameter when input to the database. The 'trp_sanitize_string' function only check the "<script></script>" with the preg_replace, the attacker can use the HTML Tag to execute javascript. 

## Vulnerable Code: 
File: includes/functions.php
Line : 151-170

```
function trp_sanitize_string( $filtered ){
	$filtered = preg_replace( '/<script\b[^>]*>(.*?)<\/script>/is', '', $filtered );

	// don't remove \r \n \t. They are part of the translation, they give structure and context to the text.
	//$filtered = preg_replace( '/[\r\n\t ]+/', ' ', $filtered );
	$filtered = trim( $filtered );

	$found = false;
	while ( preg_match('/%[a-f0-9]{2}/i', $filtered, $match) ) {
		$filtered = str_replace($match[0], '', $filtered);
		$found = true;
	}

	if ( $found ) {
		// Strip out the whitespace that may now exist after removing the octets.
		$filtered = trim( preg_replace('/ +/', ' ', $filtered) );
	}

	return $filtered;
}

```


## Step To Reproduce: 
1. Go to http://localhost:8888/wordpress/?trp-edit-translation=true
2. Input Gettext String
3. Input the payload such as "<img src=x onerror=alert(4)>"
4. Save, The payload will be executed.
5. Look on the homepage will be affected.

Video : https://drive.google.com/file/d/1PnvjHuKCvjmom6xz_sxNLBu3jixCiHy_/view?usp=sharing


## Impact:
Stored XSS on the __translated__ parameter and can be affected to various user page. 

## Fix:
Filter the __translated__ parameter with filter function. 

## Fix Code, and already tested :

Not sure if this is the best way to filter, but there's my opinion :
File: includes/functions.php
Line : 151-170

```
function trp_sanitize_string( $filtered ){
	$filtered = preg_replace( '/[^A-Za-z0-9\-]/', ' ', $filtered );

	// don't remove \r \n \t. They are part of the translation, they give structure and context to the text.
	//$filtered = preg_replace( '/[\r\n\t ]+/', ' ', $filtered );
	$filtered = trim( $filtered );

	$found = false;
	while ( preg_match('/%[a-f0-9]{2}/i', $filtered, $match) ) {
		$filtered = str_replace($match[0], '', $filtered);
		$found = true;
	}

	if ( $found ) {
		// Strip out the whitespace that may now exist after removing the octets.
		$filtered = trim( preg_replace('/ +/', ' ', $filtered) );
	}

	return $filtered;
}
```

Or 

```
function trp_sanitize_string( $filtered ){

	return esc_attr($filtered);

}
```

## How Vendor Mitigation The Issue :

Fixed Version 2.0.9
File: includes/functions.php
Line : 151-170

```
function trp_sanitize_string( $filtered ){
	$filtered = preg_replace( '/<script\b[^>]*>(.*?)<\/script>/is', '', $filtered );

	// don't remove \r \n \t. They are part of the translation, they give structure and context to the text.
	//$filtered = preg_replace( '/[\r\n\t ]+/', ' ', $filtered );
	$filtered = trim( $filtered );

	$found = false;
	while ( preg_match('/%[a-f0-9]{2}/i', $filtered, $match) ) {
		$filtered = str_replace($match[0], '', $filtered);
		$found = true;
	}

	if ( $found ) {
		// Strip out the whitespace that may now exist after removing the octets.
		$filtered = trim( preg_replace('/ +/', ' ', $filtered) );
	}

    return trp_wp_kses( $filtered );
}

function trp_wp_kses($string){
    if ( apply_filters('trp_apply_wp_kses_on_strings', true) ){
        $string = wp_kses_post($string);
    }
    return $string;
}
```

## Timeline
06 August 2021 - Report Through WPScan
06 August 2021 - WPScan verified the issue and CVE-2021-24610 has been reserved for it
06 August 2021 - Vendor contacted
09 August 2021 - Waiting for patch
24 August 2021 - Fixed & Scheduled disclosure
30 August 2021 - Approved and added to WPScan database.

Created this page is on 06 - August 2021