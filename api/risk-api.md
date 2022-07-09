# Risk API

{% hint style="warning" %}
Any source code provided on this site is unaudited and provides no guarantee that it will be functional or safe to use. Ensure you test before deploying to mainnet or production use.
{% endhint %}

## signata-risk-api

### Deployment

The Risk API is a python flask service using [Supabase](https://github.com/cl-tim/signata-docs/blob/main/docs/source/supabase.com) for record retrieval and storage. It can be easily deployed on services like [DigitalOcean](https://m.do.co/c/7802e11be119) with the following environment variables set:

`SUPABASE_URL`

`SUPABASE_KEY`

The Congruent Labs Risk API service is hosted at `https://risk.signata.net/`

### Get Risk Level

Request:

`GET /api/v1/riskLevel/{addr}`

Response:

Just responding with single values in the body for consumption by oracles.

`0 - unknown risk level`

`1...5 - risk levels`

Errors:

`400 - Invalid Address`

### Add Risk Event

Request:

`POST /api/v1/riskEvent`

Response:

`200 OK`

Errors:

`400 - Invalid Address`
