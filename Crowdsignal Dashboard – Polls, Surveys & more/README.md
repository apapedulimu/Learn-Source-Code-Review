# Crowdsignal Dashboard – Polls, Surveys & more - Privilege Escalation: Non Admin Roles Can Changes The Rating Settings

## About The Finding
Date: 07-10-2022  
Exploit Author: Nosa Shandy (Apapedulimu)  
Vendor Homepage: https://crowdsignal.com/  
Software Link: https://wordpress.org/plugins/polldaddy/  
Original Report: https://hackerone.com/reports/1725143  
PatchStack: https://patchstack.com/database/vulnerability/polldaddy/wordpress-crowdsignal-dashboard-plugin-3-0-9-privilege-escalation-vulnerability  
CVE: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-45069   

## About the Plugin:
Version: 3.0.9  
Author: Automattic, Inc.  
Requires WordPress Version: 5.5 or higher  
Requires PHP Version: 5.6 or higher  
Active Installations: 90,000+  
URL: https://wordpress.org/plugins/polldaddy/  

## Vulnerability Detail: 
## Description: 
The plugin does not implement a proper privilege to non-admin users. With this behaviour, non-admin users Can Changes The Rating Settings

## Read Code As Documentation : 

This line of code mention that the Rating Settings menu is supposed to be admin only menu.  
File: polldaddy.php  
Line : 4298 - 4316  

```
<div class="wrap">
			<?php if ( $this->is_admin ) : ?>
			<h2 id="polldaddy-header"><?php printf( __( 'Rating Results <a href="%s" class="add-new-h2">Settings</a>', 'polldaddy' ), esc_url( 'options-general.php?page=ratingsettings' ) ); ?></h2>
			<?php else : ?>
			<h2 id="polldaddy-header"><?php _e( 'Rating Results', 'polldaddy' ); ?></h2>
			<?php endif; ?>
			<div class="clear"></div>
			<form method="post" action="">
				<div class="tablenav">
					<div class="alignleft actions">
						<?php if ( $this->is_editor ) { ?>
						<select name="action">
							<option selected="selected" value=""><?php _e( 'Actions', 'polldaddy' ); ?></option>
							<option value="delete"><?php _e( 'Delete', 'polldaddy' ); ?></option>
						</select>
						<input type="hidden" name="id" id="id" value="<?php echo (int) $rating_id; ?>" />
						<input class="button-secondary action" type="submit" name="doaction" value="<?php _e( 'Apply', 'polldaddy' ); ?>" />
						<?php wp_nonce_field( 'action-rating_bulk' ); ?>
						<?php } ?>
```

## Vulnerable Code 
However, on the other source code, there's no check that the menu is only for admin.  
File: polldaddy.php  
Line : 1684 - 1711  

```
	function settings_page() {
		global $page, $action;
		?>
		<div class="wrap" id="manage-polls">
			<div class="cs-wrapper">
				<?php
				$this->set_api_user_code();

				if ( isset( $_GET['page'] ) ) { // phpcs:ignore
					$page = $_GET['page']; // phpcs:ignore
				}
				if ( 'crowdsignal-settings' === $page ) {
					if ( ! $this->is_author ) { // check user privileges has access to action.
						return;
					}
					$this->plugin_options();
				} elseif ( 'ratingsettings' === $page ) {
					if ( 'update-rating' === $action ) {
						$this->update_rating();
					}

					$this->rating_settings();
				}
				?>
			</div>
		</div>
		<?php
	}
```

## Step To Reproduce :
1. Login as Author / Contributor
2. Navigate to `ratingsettings` page http://{wordpress}/wp-admin/options-general.php?page=ratingsettings&rating=comments
3. You will be enable to make any changes with the lower roles rather than admin.

Note:  
The menu is also shown on the Settings menu dropdown

Video:  
https://drive.google.com/file/d/1mGmDHMbl5_Hcv_bB-YhlrYfTaYOoM1WE/view?usp=share_link

## How Vendor Mitigation The Issue :

https://github.com/Automattic/crowdsignal-plugin/pull/89  
Fixed Version 3.0.10  
File: polldaddy.php  
Line : 205  

```
		foreach( array( 'crowdsignal-settings' => __( 'Crowdsignal', 'polldaddy' ), 'ratingsettings' => __( 'Ratings', 'polldaddy' ) ) as $menu_slug => $page_title ) {
			// translators: %s placeholder is the setting page type (Poll or Rating).
			$settings_page_title = sprintf( esc_html__( '%s', 'polldaddy' ), $page_title );
			$hook = add_options_page( $settings_page_title, $settings_page_title, $menu_slug == 'ratingsettings' ? 'manage_options' : $capability, $menu_slug, array( $this, 'settings_page' ) );
			add_action( "load-$hook", array( $this, 'management_page_load' ) );
		}
```

## Timeline
07 October 2022 - Report Through Hackerone  
07 October 2022 - First Response  
03 November 2022 - Triaged  
07 November 2022 - Resolved  
16 November 2022 - 100$ Bounty  