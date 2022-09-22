# Identity Provider Design

auth.signata.net

## Overview

<figure><img src="../.gitbook/assets/2.png" alt=""><figcaption></figcaption></figure>

## Authentication



## Authorization



## Integration



## Keycloak Configuration

### Create Realm

Click **Add Realm**.

![](<../.gitbook/assets/image (24) (1).png>)

Set the name to **Signata** and click **Create**.

![](<../.gitbook/assets/image (4) (2) (2).png>)

### Configure Authentication

Click **Authentication** in the left menu.

![](<../.gitbook/assets/image (17) (2).png>)

In Flows, click **New**.

![](<../.gitbook/assets/image (7).png>)

Set the Alias to **Signata** and click **Save**.

![](<../.gitbook/assets/image (10) (1).png>)

Click **Add execution**.

![](<../.gitbook/assets/image (1) (3).png>)

Select **Signata Signature** and click **Save**.

![](<../.gitbook/assets/image (15).png>)

Set the Signata Signature to **REQUIRED**. Under Actions click **Config**.

![](<../.gitbook/assets/image (3) (2).png>)

Set the Alias to **Signata**, the Infura Id and Node URLs to the keys provided by Infura. Set the Timeout to **120**. Click **Save**.

![](<../.gitbook/assets/image (14) (2).png>)

### Create Client

Click on **Clients** in the left menu.

![](<../.gitbook/assets/image (5) (1) (1).png>)















