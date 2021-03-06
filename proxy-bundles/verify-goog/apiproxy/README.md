# JWT Verification - token generated by Google Sign-in

This API proxy validates a JWT that was created and signed by Google Sign-in.

This proxy will work on the Apigee Edge public cloud release.

## Deploying

Several notes:

* use a tool like [apigeetool](https://github.com/apigee/apigeetool-node) or [pushapi](https://github.com/carloseberhardt/apiploy) or
[importAndDeploy.js](https://github.com/DinoChiesa/apigee-edge-js/blob/master/examples/importAndDeploy.js) 
to deploy the proxy.

* Before you deploy the proxy you need to create a cache on the
environment. The cache should be named 'cache1'.

## Invoking

To verify a token generated by Google, use this curl command:

```
  curl -i -X POST http://myorg-test.apigee.net/jwt-verify-goog/t1 -H "Authorization: Bearer $jwt"

```

This validation uses the Google certificate, which is obtained from [the well-known public endpoint](https://www.googleapis.com/oauth2/v3/certs).

To actually obtain a JWT
generated by Google, you need to sign-in using a client_id registered with
Google Sign-in. Read about that [here](https://developers.google.com/identity/protocols/OpenIDConnect).

Once you register an app with Google, here's [a page that can help you start the Google Sign-in process](http://dinochiesa.github.io/openid-connect/goog-login.html).


## Commentary

The JWT Verification policy in Apigee Edge is smart enough to extract keys from a JWK set.  Google Sign-in [exposes a JWK set](https://www.googleapis.com/oauth2/v3/certs). This endpoint provides the public keys that correspond to the private keys used by Google Sign-in to sign JWT.  Each public key is identified by a "key ID", aka "kid". The content available at this endpoint changes as Google rotates keys, but currently the response looks like this:

```json
{
 "keys": [
  {
   "kty": "RSA",
   "alg": "RS256",
   "use": "sig",
   "kid": "02bf92b561733f6f580a7038156309cbfc6b486f",
   "n": "zWQXn72L60w20bo84sujDsAIUJs8gPhrN4b_MRDfB5skhobiKbdShTz1iqW8kpAdFGk2L3hEBQFs4pHzzc3G_cZK_ZIWmGd4IZ0AL5JzyUXmjAwtowGUEmHlWltUZ2KBI-o9PjduOxMovNf7HQ2qhARp_ib12hpDcDTrIcrO9R5p-n-4zKDBnRJLTliqhxaUt232v8gyS4oVYOTVAlmoQXGHnS503XWCxx_bcNk177Y0onmejUAAK8WgN8u9e_eAoZIcApw3h4NTLXwiHtm4mJuHAEhsX__goZ6wnjCW7DW48eWoK6cbvc4X6DlWGRa4JuIaAaic80Z7lplJ7M2gRw",
   "e": "AQAB"
  },
  {
   "kty": "RSA",
   "alg": "RS256",
   "use": "sig",
   "kid": "a12c3a610919330830079925df9c95ad85274580",
   "n": "1-u8ezbZm83wJi_R7X2EqYhKyNGHXuuu0qYcDCX4ZU6FFSCiwnQzQ4K8M5JGBv67hJWjun7kZ18mhtPmfPsf7TmyiZcokxut_jW0bfeLD8NZBdy4EIYspx29qaCQ3XQaiY4FdYefbuQUx-svILOqoh0ke2imyUGBBnwby5BifjTqLia1KjAkNbBmdfpFMO5NTaFNkmC59gkQom82KxoTouL__X05TzVS177KzO5B7-NqgviQ6ZYUb2zGr2pGcxjw9I5LsgQO8aPoWsZOeJRXb9OBkNXpDGWkoyeMYHtmvYXf8l2Hmxqk6NwBoumobSmnhiUemoyHtBiAstj_7s3SCw",
   "e": "AQAB"
  },
  {
   "kty": "RSA",
   "alg": "RS256",
   "use": "sig",
   "kid": "77d4ed62ee21d157e8a237b7db3cbd8f7aafab2e",
   "n": "snfgF8zfxaHCR1NggGKTLnn04KN_czOjZIaUCmM_rWkcgUKIp2py5iGUTCK-Pp_m7BGgcTD4j5vDSPVX1235ey4AysXN__2n00DLdhJZLqtAty0oIbdDlKQo9tCrsjC1PGlCq5ac7OFYYaXDNYnm1-tind925CzBKdzC0kwsf6bCmGrcwYMae4Rhd8iRBdpIfDR9CBa8eEed_07mCj_pNlDMqjGYCe-Sp02ub3hyg19RJpsnEj_cFbOlCIC6HX4DYu5JYyYJIuYZthZmPbCa1wF3r-yOP9vjMr4P8jCcmHwFJnHBqOVYwSt1NRa-goR4JzflTW58GN1LynN8DhsPEQ",
   "e": "AQAB"
  }
 ]
}
```

The API Proxy uses a ServiceCallout to perform a GET on that URL, and then caches the result for one hour. Then, verification can be configured like this:

```xml
<VerifyJWT name="Verify-JWT-1">
    <Algorithm>RS256</Algorithm>
    <Source>inbound.jwt</Source>
    <PublicKey>
      <JWKS ref='cached.google.jwks'/>
    </PublicKey>
    <Issuer>https://accounts.google.com</Issuer>
</VerifyJWT>
```

Simple!
