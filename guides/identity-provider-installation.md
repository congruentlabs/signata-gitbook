# Identity Provider Installation

{% embed url="https://m.do.co/c/7802e11be119" %}

{% hint style="warning" %}
Some of this documentation is intentionally vague to not reveal production deployment information. If you wish to deploy your own Signata Identity Provider, contact support for assistance.
{% endhint %}

{% hint style="info" %}
The links on this page to DigitalOcean are referral links - you can help support the Signata project by using those links to sign up and try DigitalOcean!
{% endhint %}

## Compiling Keycloak

The Signata IdP is not a standard version of keycloak. It is the containerized version of keycloak with a custom Signata authentication provider injected during build.

If you're compiling for the Signata production instance, you can leave the build script as-is. If you're compiling for a different instance (like your own fork), then you will need to modify the build PowerShell scripts to tag the container with the name you wish to use.

```powershell
cd keycloak-signata-extension/scripts/
./build-production.ps1
```

If you're using a DigitalOcean docker registry, you will need to log into it first using an API key generated from the DO interface:

```
doctl auth init
doctl registry login
```

You can alternatively use docker hub to host the container image, but instructions for that won't be provided here.

## Deploying Keycloak

### Postgres

Create a new **Database Cluster** on DigitalOcean. Select your desired datacenter region, and select **PostgreSQL** **v14**.

Once provisioned, add a new database called **keycloak**.

![](<../.gitbook/assets/image (2).png>)

Add a new user also called **keycloak**.

![](<../.gitbook/assets/image (4) (2).png>)

Once you've deployed the DigitalOcean App in later steps, edit the database to add it as a **Trusted Source** to limit network traffic to only the keycloak app.

![](<../.gitbook/assets/image (5) (1).png>)

### Container Registry

Create a container registry to host the built docker images for the keycloak app.

![](<../.gitbook/assets/image (8) (3).png>)

In the keycloak-signata-extension repository, use PowerShell to run **scripts/build\_production.ps1**. This will compile the container, and push the image to your registry.

### DigitalOcean App

{% embed url="https://m.do.co/c/7802e11be119" %}

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

Make sure **HTTP Port** in the App configuration is set to **8080**. This should be sufficient to build and host the container. Don't set the port in the environment variables as that seems to prevent the admin portal from functioning correctly.

The keycloak instance will just listen on HTTP. As DO performs the traffic proxying, it will handle all TLS configuration for the public endpoint.

If your deployment fails and you're getting SQL errors, make sure the public URL for your database is correct as well as the port. DO postgres **doesn't** use the standard 5432 port.
