# ansible-freeradius

Ansible role for the installation of FreeRADIUS in the role of IdP & SP, SP only, or as a proxy for eduroam.cz. Successful use presupposes a solid knowledge of Linux administration and at least a basic knowledge of automated management using ansible.com.

The role supports *RadSec*, *Operator-Name*, *Chargeable-User-Identity*, *enforcing match of inner and outer identity*. Furthermore, it allows non-EAP authentication only to local realms (for captive portals, if you really have to use them).

I assume that you will insert the ansible-freeradius role into your own project. For single-purpose testing, just follow the following instructions. Create any directory and execute:

\```shell
git clone https://github.com/CESNET/ansible-freeradius.git roles/freeradius
mkdir -p host_vars group_vars files/certs
cp roles/freeradius/examples/playbook-freeradius.yml .
cp roles/freeradius/examples/inventory.conf .
cp roles/freeradius/examples/ansible.cfg .
cp roles/freeradius/examples/chain_TERENA_SSL_CA_3.pem files/certs/
cp roles/freeradius/examples/chain_CESNET_CA4.pem files/certs/
\```

You must edit the `inventory.conf` file to point to your server. You need to create a `host_vars/your-radius.realm.cz.yml` file, as a model use the files [explanatory notes on content](./Parameters.md):

- [semik-dev.cesnet.cz-IdPSP-PKCS12.yml](https://github.com/CESNET/ansible-freeradius/blob/master/examples/semik-dev.cesnet.cz-IdPSP-PKCS12.yml) for IdP & SP with a certificate in PKCS#12 format
- [semik-dev.cesnet.cz-IdPSP.yml](https://github.com/CESNET/ansible-freeradius/blob/master/examples/semik-dev.cesnet.cz-IdPSP.yml) for IdP & SP with a certificate in the more common PEM format, users in LDAP with an eduroam password different from the main password in LDAP
- [semik-dev.cesnet.cz-IdPSP-msAD.yml](https://github.com/CESNET/ansible-freeradius/blob/master/examples/semik-dev.cesnet.cz-IdPSP.yml) for IdP & SP with a certificate in the more common PEM format, users in MS AD and authentication using NTLMv1, addition to the domain must be done manually, ansible does not do this, see [instructions](https://www.eduroam.cz/cs/spravce/pripojovani/radius/freeradius3/windowsad).
- [semik-dev.cesnet.cz-SP.yml](https://github.com/CESNET/ansible-freeradius/blob/master/examples/semik-dev.cesnet.cz-SP.yml) for SP only installation
- [semik-dev.cesnet.cz-proxy.yml](https://github.com/CESNET/ansible-freeradius/blob/master/examples/semik-dev.cesnet.cz-proxy.yml) proxy mode when the home realm is passed to another RADIUS server

Confidential server configuration information is encrypted in the vault, for example:

\```shell
ldap:
  eduroam:
    bindPass: "{{ semik_dev_cesnet_cz.ldap_passwd }}"
\```

Create `group_vars/idp_vault.yml` with the following content, just change the first line to your hostname (**dots and possible hyphens must be replaced with underscores, i.e., `semik-dev.cesnet.cz` will be `semik_dev_cesnet_cz`**):

\```shell
semik_dev_cesnet_cz:
  ermon_secret: shared password to ermon.cesnet.cz
  ldap_passwd: password to LDAP account
  radsec_key_password: password to private key for radsec
  eap_key_password: password to private key for eap
\```

Configuration information is divided into two files to separate confidential information, which is worth encrypting using ansible-vault, and the fairly public ones. In addition, I assume that the encrypted `group_vars/idp_vault.yml` file shares confidential information from other servers and possibly other roles.

## Certificates

In case of using PKCS#12 format, the role assumes that the RADIUS server uses the same certificate for the connection with the national RADIUS server as for the (optional) role of IdP. This requirement comes from sharing part of the code with the playbook for Shibboleth IdP (eduID.cz), where it is appropriate and in addition, such an encrypted certificate can be stored in the public GIT without fear of misuse. The file must be located in `files/semik-dev.cesnet.cz.p12`, or a suitably named file.

FreeRADIUS uses the PEM format for certificates, in this case, it expects them in `files/certs/hostname.{crt,key}`, different certificates can be used for RadSec and EAP, which will probably be a frequent case.

The role also works with certificates for verifying the trust of the LDAP server. They are linked in `host_vars/semik-dev.cesnet.cz.yml` in the `ldap.CAChain` variable.

It also works with certificates for verification of the superior RADIUS server. It is referred to in `eduroam.topRADIUS.CAChain`. The role assumes that the superior RADIUS is not `radius1.eduroam.cz`, but a general superior RADIUS.

## Launch

\```shell
export ANSIBLE_INVENTORY=./inventory.conf
ansible-playbook playbook-freeradius.yml 
\```
