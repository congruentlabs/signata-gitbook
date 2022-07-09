# Risk API

{% hint style="warning" %}
Any source code provided on this site is unaudited and provides no guarantee that it will be functional or safe to use. Ensure you test before deploying to mainnet or production use.
{% endhint %}

## signata-risk-api

{% embed url="https://github.com/congruentlabs/signata-risk-api" %}

The Risk API is a python flask service using [Supabase](https://github.com/cl-tim/signata-docs/blob/main/docs/source/supabase.com) for record retrieval and storage.

{% embed url="https://supabase.com" %}

It can be easily deployed on services like [DigitalOcean](https://m.do.co/c/7802e11be119) with the following environment variables set:

`SUPABASE_URL`

`SUPABASE_KEY`

{% embed url="https://m.do.co/c/7802e11be119" %}

{% swagger method="get" path="/riskLevel" baseUrl="https://risk.signata.net/api/v1" summary="Get Risk Level" %}
{% swagger-description %}

{% endswagger-description %}

{% swagger-parameter in="path" name="addr" required="true" %}

{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```javascript
0 - unknown risk level
1..5 - risk levels
```
{% endswagger-response %}

{% swagger-response status="400: Bad Request" description="" %}
```javascript
Invalid Address
```
{% endswagger-response %}
{% endswagger %}

{% swagger method="post" path="/riskEvent" baseUrl="https://risk.signata.net/api/v1" summary="Add Risk Event" %}
{% swagger-description %}
Injects a risk event into the database.

Requires a 'write' x-api-key
{% endswagger-description %}

{% swagger-parameter in="body" name="address" required="true" %}

{% endswagger-parameter %}

{% swagger-parameter in="body" name="geolocation_thumbprint" %}

{% endswagger-parameter %}

{% swagger-parameter in="body" name="access_type" %}

{% endswagger-parameter %}

{% swagger-parameter in="body" name="reported_by" %}

{% endswagger-parameter %}

{% swagger-parameter in="body" name="device_thumbprint" %}

{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```javascript
Event Added
```
{% endswagger-response %}

{% swagger-response status="400: Bad Request" description="" %}
```javascript
Invalid Address
```
{% endswagger-response %}
{% endswagger %}
