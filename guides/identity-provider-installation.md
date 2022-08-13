# Identity Provider Installation

## Keycloak Deployment

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.cdn.digitaloceanspaces.com/WWW/Badge%201.svg)](https://www.digitalocean.com/?refcode=7802e11be119\&utm\_campaign=Referral\_Invite\&utm\_medium=Referral\_Program\&utm\_source=badge)

{% hint style="warning" %}
Some of this documentation is intentionally vague to not reveal production deployment information. If you wish to deploy your own Signata Identity Provider, contact support for assistance.
{% endhint %}

{% hint style="info" %}
The links on this page to DigitalOcean are referral links - you can help support the Signata project by using those links to sign up and try DigitalOcean!
{% endhint %}

### Postgres

Create a new **Database Cluster** on DigitalOcean. Select your desired datacenter region, and select **PostgreSQL** **v14**.

Once provisioned, add a new database called **keycloak**.

![](<../.gitbook/assets/image (2).png>)

Add a new user also called **keycloak**.

![](<../.gitbook/assets/image (4).png>)

Once you've deployed the DigitalOcean App in later steps, edit the database to add it as a **Trusted Source** to limit network traffic to only the keycloak app.

![](<../.gitbook/assets/image (5).png>)

### Container Registry

Create a container registry to host the built docker images for the keycloak app.

![](<../.gitbook/assets/image (8).png>)

In the keycloak-signata-extension repository, use PowerShell to run **scripts/build\_production.ps1**. This will compile the container, and push the image to your registry.

### DigitalOcean App

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.cdn.digitaloceanspaces.com/WWW/Badge%201.svg)](https://www.digitalocean.com/?refcode=7802e11be119\&utm\_campaign=Referral\_Invite\&utm\_medium=Referral\_Program\&utm\_source=badge)

Create a new DigitalOcean App, using your container registry and the keycloak image you wish to deploy.

Set the following environment variables on the component:

| Key                               | Value                                           | Info                                                                     |
| --------------------------------- | ----------------------------------------------- | ------------------------------------------------------------------------ |
| KC\_DB\_URL                       | jdbc:postgresql://{public\_url}:{port}/keycloak | Obtain {public\_url} and {port} from your managed database configuration |
| KC\_DB\_USERNAME                  | keycloak                                        |                                                                          |
| KC\_DB\_PASSWORD                  | {password}                                      | Get the password from your managed database configuration                |
| KEYCLOAK\_ADMIN                   | {username}                                      | Set {username} to a username you want to use                             |
| KEYCLOAK\_ADMIN\_PASSWORD         | {password}                                      | Set {password} to a strong password                                      |
| KC\_PROXY                         | edge                                            |                                                                          |
| KC\_HTTP\_ENABLED                 | true                                            |                                                                          |
| KC\_HOSTNAME                      | {url}                                           | Set {url} to the public URL that your instance will be hosted at         |
| KC\_HOSTNAME\_STRICT\_BACKCHANNEL | true                                            |                                                                          |
| KC\_HOSTNAME\_STRICT              | true                                            |                                                                          |

Make sure **HTTP Port** in the App configuration is set to **8080**. This should be sufficient to build and host the container. Don't set the port in the environment variables as that seems to prevent the admin portal from functioning.

The keycloak instance will just listen on HTTP. As DO performs the traffic proxying, it will handle all TLS configuration for the public endpoint.

If your deployment fails and you're getting SQL errors, make sure the public URL for your database is correct as well as the port. DO postgres doesn't use the standard 5432 port.
