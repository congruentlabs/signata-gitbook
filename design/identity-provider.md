# Identity Provider

auth.signata.net

## Overview



## Authentication



## Authorization



## Integration



## Keycloak Configuration

### Create Realm

Click **Add Realm**.

![](<../.gitbook/assets/image (24).png>)

Set the name to **Signata** and click **Create**.

![](<../.gitbook/assets/image (4).png>)

### Configure Authentication

Click **Authentication** in the left menu.

![](<../.gitbook/assets/image (17).png>)

In Flows, click **New**.

![](<../.gitbook/assets/image (7).png>)

Set the Alias to **Signata** and click **Save**.

![](<../.gitbook/assets/image (10).png>)

Click **Add execution**.

![](<../.gitbook/assets/image (1).png>)

Select **Signata Signature** and click **Save**.

![](<../.gitbook/assets/image (15).png>)

Set the Signata Signature to **REQUIRED**. Under Actions click **Config**.

![](<../.gitbook/assets/image (3).png>)

Set the Alias to **Signata**, the Infura Id and Node URLs to the keys provided by Infura. Set the Timeout to **120**. Click **Save**.

![](<../.gitbook/assets/image (14).png>)

### Create Client

Click on **Clients** in the left menu.

![](<../.gitbook/assets/image (5).png>)















