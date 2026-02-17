
This part of documentation is a explanation to all the configurations that are being made in **createClient()** function inside the **drupal** script.

| Command                                                                     | Description                                                                                                                                         |
| --------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `vendor/bin/drush config:set openid_connect.settings.keycloak enabled true` | Enables the Keycloak OpenID Connect provider in Drupal.                                                                                             |
| `settings.client_id '$CLIENT_ID'`                                           | Sets the Keycloak client ID that Drupal will use for authentication.                                                                                |
| `settings.client_secret '$CLIENT_SECRET'`                                   | Sets the secret key used to authenticate Drupal with Keycloak.                                                                                      |
| `settings.keycloak_base '$KEYCLOAK_BASE'`                                   | Defines the base URL of your Keycloak server, you may have provided when asked.                                                                     |
| `settings.keycloak_realm '$KEYCLOAK_REALM'`                                 | Specifies the Keycloak realm Drupal should connect to. (*eg. master*)                                                                               |
| `settings.keycloak_sso true`                                                | Enables single sign-on (SSO). If a user is already logged into Keycloak, they will be automatically logged into Drupal without seeing a login form. |
| `vendor/bin/drush cr`                                                       | Clears Drupal’s cache to apply all configuration changes.                                                                                           |
