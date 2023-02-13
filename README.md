# moodle-matrix-docker:
A docker environment for Matrix + Keyloak + Element for Moodle developers

# Overview
This is mostly based on https://github.com/mattporritt/moodle-docker

The services set up are:
* Mailhog
* Synapse
* Element
* Keycloak

Once the following steps are complete the sites can be accessed at the following URLs:
* Keycloak: https://keycloak:8443/
* Element: https://element:8081/
* Synapse: https://synapse:8008/
* Mailhog: http://localhost:1234/_/mail

## Host file
To make it easier to access the sites that have been setup and to allow correct SSO workflows, the hosts file on the machine running the docker containers needs to be updated. With extra services added to the localhost entry.

Update your local hosts file: `/etc/hosts` to include the following names for the localhost IP (127.0.0.1):
* Keycloak
* Synapse
* Element
* Webserver (Moodle)

If you haven’t already customised your hosts file the line should look like:<br/>
`127.0.0.1   	localhost synapse webserver keycloak element`


## SSL Setup
Next make the self signed certs so we can run Moodle and associated services over ssl/tls. This is required to setup SSO.<br/>
There is a helper script that creates the root CA and dev certificates and keys.

Change to the directory of the script first:<br/>
`cd moodle-matrix-docker/certs`

Then run it for each service that we require certificates for. The first time the script is run it will generate a root cert and key. After that it will re-use the root cert and just make the certs required for each service we want to run using SSL/TLS.

Then run it for each additional service we need<br/>
* `./createcerts.sh keycloak`
* `./createcerts.sh synapse`
* `./createcerts.sh element`

These certs will be automatically loaded into the containers. We only need to generate them.

## Build and install
Next we need to build our version of the moodle dev container:<br/>
`cd moodle-matrix-docker`<br/>
`docker-compose up -d`

## Synapse admin user
We also need to create an initial Synapse admin user. This is an initial user you'll need before you can use the Matrix API.<br/>
Synapse provides a cli tool to create users. To create an initial admin user:<br/>
`cd moodle-docker/bin`<br/>
```
docker exec -it {containerid} register_new_matrix_user \
-u admin \
-a \
-c /data/homeserver.yaml \
https://synapse:8008/
```
Where `{containerid}` is the Docker ID of the running Synapse container.

# Keycloak IdP Setup
This will set up Keycloak as an Identity Provider (IdP) for Moodle and Synapse. Users will be able to log into Moodle and Element via Keycloak once the following configuration is complete.

## Keycloak
The following steps will set up a Moodle LMS client in Keycloak so users can authenticate to Moodle from Keycloak using OIDC/Oauth.

Access the Moodle realm in Keycloak:<br/>
* First log into Keycloak (https://keycloak:8443/ ) using the admin credentials you defined in the .env file in the root of the moodle-matrix-docker project.
* Then click on the link to Keycloak Administration
* Then on the left of the screen change the “Realm Select” drop down menu from master to moodle.

Set up the Moodle client:<br/>
* Click Clients from the Manage menu on the left of the page
* From the list of clients that are displayed click on the “moodle-client” link in the Client ID column
* Click the Credentials tab
* Click the Regenerate button for the Client secret
* Note the Client secret.

Set up the Synapse client:<br/>
* Click Clients from the Manage menu on the left of the page
* From the list of clients that are displayed click on the “moodle-client” link in the Client ID column
* Click the Credentials tab
* Click the Regenerate button for the Client secret
* Note the Client secret.

We also need to create at least one user in the moodle realm in keycloak. All users that log into Moodle using SSO via Keycloak need an account in the moodle realm.<br/>
To do this:<br/>
* Click Users from the Manage menu on the left of the page
* Click the Add user button
* Set the following settings for the new user:
  - Username
  - Email (can be fake)
  - Set Email verified to true/on
  - First name
  - Last name
* Click the create button