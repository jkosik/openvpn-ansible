# Deployment of OpenVPN server

**Assumptions**
* VM is created in Openstacki (etsted in Openstack with Ubuntu 16 images).
* VM is dualhomed. *It is recommended to attach external interface to the VM first to set default GW properly.*
* External interface will act as VPN listener and will forward traffic to network where internatl interface is conencted to.
* Routed mode used as default

# Playbook
**To customize playbook it is needed to check global_vars.**
The main role of the playbook is to install and configure OpenVPN server including first initial user account as defined in group_vars/all.yml.
All connection files needed for the initial user are fetched to /fetched:
* ca.crt
* init_client.key
* init_client.crt
* client.ovpn

*client.ovpn* is a script delivered to make connection as easy as possible - see below **Client connection**. 
Client.ovpn for initial user is pushed from ansible-openvpn role. For subsequent user creation (create_user.yml) the client.ovpn template part of the own playbook structure.

### Running the playbook
```
$ git clone <this repo>
$ ansible-playbook -i inventory main.yml
```

### inventory file
```
[all]
10.235.3.0 inv_hostname="openvpn-test"
```
### Required roles
* ansible-py2-bootstrap - installs python2.7 (mandatory)
* ansible-fqdn - updates hostname (if needed)
* ansible-set-proxy - set proxy (if needed)
* ansible-openvpn

### Client connection
```
cd fetched/
sudo openvpn --config client.ovpn
```

# Certificate lifecycle
```create_user.yml``` and ```revoke_user.yml``` support certificate/user lifecycle.
Both files have own variable in group_vars/all.yml which has to be updated based on needed action - what to create, what to revoke.
Neither of the these two playbooks needs ansible-openvpn. Generated client.ovpn template for new users is part of the playbook directory structure. 
*Client.ovpn template for initial user is part of ansible-openvpn role structure.*

### create_user.yml
Create only one user per playbook run. The reason is need for running custom bash scripts calling openvpn tools and pushing Ansible variables there.
In case additional users are needed, change variable ```client``` in ```group_vars/all.yml``` and run the script again.
**Users must be unique due to behaviour of OpenVPN database.** Creation of already existing user name will lead to failure.
Duplicate user names are allowed only once the first account with the same name was revoked. New account will be differentiated based on new ID. See example for *juraj2* account:
```
root@openvpn-test:/home/ubuntu/openvpn-ca# cat keys/index.txt
V	270223222639Z		01	unknown	/C=SK/ST=BA/L=Bratislava/O=juraj_org/OU=juraj_ou/CN=juraj/name=JURAJ-KEY/emailAddress=juraj@abc.com
V	270223222658Z		02	unknown	/C=SK/ST=BA/L=Bratislava/O=juraj_org/OU=juraj_ou/CN=juraj/name=JURAJ-KEY/emailAddress=juraj@abc.com
R	270223222924Z	170225223011Z	03	unknown	/C=SK/ST=BA/L=Bratislava/O=juraj_org/OU=juraj_ou/CN=juraj2/name=JURAJ-KEY/emailAddress=juraj@abc.com
V	270223223723Z		05	unknown	/C=SK/ST=BA/L=Bratislava/O=juraj_org/OU=juraj_ou/CN=juraj2/name=JURAJ-KEY/emailAddress=juraj@abc.com
```

### revoke_user.yml
Revoke only one user per playbook run. The reason is need for running custom bash scripts calling openvpn tools and pushing Ansible variables there.
In case user has to be revoked, change variable ```revoke_client``` in ```group_vars/all.yml``` and run the script.
Revocation of already revoked user account makes no problem.





