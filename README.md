# Description

This Vagrantfile creates a full Chef Automate Environment (Chef Server, Delivery Server and a Build Node).

# Usage
Clone the repository. Generate or use your own certificates (id_rsa, id_rsa.pub), put them into the folder, add a Delivery license and the Delivery binaries as well and start it using "vagrant up".
Keep in mind that IP addresses and binaries are fixed, you have to change them to your version and you preferred IP address.

## Nodes

* automate      192.168.100.10
* chef-server   192.168.100.20
* buildnode     192.168.100.30
* compliance    192.168.100.40

Please note that you can access the Delivery Server only by name (etc/hosts) and it´s public IP from external.

## Logins

You can specifiy the account details for the Chef Server and the Chef Automate Enterprise Name in the account.yml.

* chef-server
  * username = delivery
  * password = master

* Automate Server
Look at the vagrant output for the admin and builder password.

e.g.
```
==> Automate Server: Created enterprise: cjohannsen
==> Automate Server: Admin username: admin
==> Automate Server: Admin password: jcImgI3kf0aS3iBeaD6ggYw5abyqoNB2DYw=
==> Automate Server: Builder Password: Mf7R4Pg5i9wWktGM/oJGRdAIW7/WgSXSpF4=
==> Automate Server: Web login: https://automate/e/cjohannsen/
```

## Comments

Feel free to commit.

## version

Version 0.7

## Author
Christian Johannsen, 2016
