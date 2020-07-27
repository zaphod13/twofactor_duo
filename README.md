# twofactor_duo
Experimental Duo two-factor auth provider for Nextcloud

## Configuration
Place files & sub-directories for this project into directory named *twofactor_duo* in the *apps* folder of your NextCloud installation *(i.e. /var/www/html/apps)*

Login to the Duo admin console and add a Web SDK application for the NextCloud logins. Make note of the Integration Key (IKEY), the Secret Key (SKEY), and the API Hostname (HOST).

Add your duo configuration to your Nextcloud's `config/config.php` fils (for AKEY use IKEY value):
```
'twofactor_duo' => [
    'IKEY' => 'xxx',
    'SKEY' => 'yyy',
    'HOST' => 'zzz',
    'AKEY' => '123',
  ],
```
## Nextcloud server patch
The app provides a custom CSP which the Nextcloud server currently does not support. The following patch adds this customization support:
```patch
 core/Controller/TwoFactorChallengeController.php   | 12 ++++++--
 .../TwoFactorAuth/IProvidesCustomCSP.php           | 33 ++++++++++++++++++++++
 2 files changed, 42 insertions(+), 3 deletions(-)

diff --git a/core/Controller/TwoFactorChallengeController.php b/core/Controller/TwoFactorChallengeController.php
index fd4811d3ff..ed4c4f45d4 100644
--- a/core/Controller/TwoFactorChallengeController.php
+++ b/core/Controller/TwoFactorChallengeController.php
@@ -1,4 +1,5 @@
 <?php
+
 /**
  * @copyright Copyright (c) 2016, ownCloud, Inc.
  *
@@ -29,6 +30,7 @@ use OC_Util;
 use OCP\AppFramework\Controller;
 use OCP\AppFramework\Http\RedirectResponse;
 use OCP\AppFramework\Http\TemplateResponse;
 use OCP\Authentication\TwoFactorAuth\TwoFactorException;
 use OCP\IRequest;
 use OCP\ISession;
@@ -135,7 +137,11 @@ class TwoFactorChallengeController extends Controller {
 			'redirect_url' => $redirect_url,
 			'template' => $tmpl->fetchPage(),
 		];
-		return new TemplateResponse($this->appName, 'twofactorshowchallenge', $data, 'guest');
+		$response = new TemplateResponse($this->appName, 'twofactorshowchallenge', $data, 'guest');
+		if ($provider instanceof IProvidesCustomCSP) {
+			$response->setContentSecurityPolicy($provider->getCSP());
+		}
+		return $response;
 	}
 
 	/**
 
```
## Notes
If there are any issues, check /var/log/httpd/error_log to see what errors are being thrown

If used with LDAP/Active Directory auth, you will need to change LDAP auth to use actual username not UUID.  Do this by going into LDAP integration settings, in Expert section set the Internal Username Attribute to sAMAccountName






