# Drupal and Keycloak on seperate VMs

## 1. Setup VM for both Drupal and Keycloak

Refer **'Seprated VM Setup/Setting Up Host-only Adapter'**, it covers VM setup followed by a proper Host-Only network connection that allows us to get a static ip and is useful in SSO setup.

## 2. Setup drupal and keycloak VM

Install keycloak and drupal on respective VMs, Refer **'Seprated VM Setup/Server A - Keycloak Setup'** and **'Seprated VM Setup/Server B - Openplc website setup'**

## 3. Keycloak script

Create dir and set permissions

```
mkdir ~/keycloak
cd keycloak
```

Create the following files with given contents

config.json

```json
[
  {
    "keycloak_realm": "",
    "keycloak_base": "",
    "keycloak_user": "",
    "keycloak_password": "",
    "drupal_base": "",
    "keycloak_client_id": ""
  }
]
```

client.json

```json
{
  "clientId": "",
  "enabled": true,
  "publicClient": false,
  "protocol": "openid-connect",
  "clientAuthenticatorType": "client-secret",
  "standardFlowEnabled": true,
  "redirectUris": [""]
}
```

Add the script

```sh
wget https://github.com/CoolSrj06/Fossee-Daily-Progress/blob/master/Automated%20SSO%20Setup/keycloak_user_config_script.md
```

Fix permissions and run the script

```sh
sudo chown -R keycloak:$USER .
sudo chmod -R g+w .
chmod +x keycloak
./keycloak
```

This will generate secret.json file copy the contents of the file

## 4. Drupal script

Create a dir and copy required files

```sh
mkdir drupal
cd drupal
```

Download script file

```sh
wget https://github.com/CoolSrj06/Fossee-Daily-Progress/blob/master/Automated%20SSO%20Setup/openplc_config_script.md

# copy the script 
```

``` bash
# We need to enter the bash of Podman container
podman exec -it <container name> bash
# once we are inside the shell of container
apt install vim -y
vi openplc_config_script
# paste the script content here then save and exit (:wq)

# you will need your shell user name
cat /etc/passwd
#note the username, alternate command is ls -al
# out will be in format eg:
# rwx---x-r-x <user> <group> filename/dir-name
# note this user
```

Run the script file

```sh
chmod +x openplc_config_script

./openplc_config_script --client-id <client-name, eg: openplc> ----client-secret <secret-credential> --base <keycloak-base-url> --realm <eg: master> --email <email-of-keycloak-user> --drupal-user <username> --drupal-password <password> ----user <shell-user>
```

## 5. Test SSO

Open your drupal site and try to login