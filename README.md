## Apigee Example Proxy - Lookup Quota Externally

This is an example API proxy, that enforces a rate limit, or quota, with the
Quota policy.  That's a pretty simple idea. The twist with this example is, the
proxy looks up the quota in an external system.

In [this screencast](https://www.youtube.com/watch?v=j3zLlp3Q8UA&t), I
demonstrate the use of this proxy.


This sample works in Apigee X. I have not tested it in Apigee Edge.  To get it
to work in Edge, at the least, you will have to make one modification: modify
the ProxyEndpoint to use a VirtualHost that exists in your environment.


## Disclaimer

This example is not an official Google product, nor is it part of an official
Google product. It's just an example.


## Setup

To use this proxy, you need to perform some initial setup.

I won't outline the EXACT set of steps, but I'll describe them for you.

1. You need an Apigee organization and environment, and you must have
   permissions to import proxies and deploy them to the environment.

2. You need a way to invoke the API proxy, probably via something like curl or if you have Powershell, `Invoke-WebRequest` . Or, via a GUI tool like Postman.

3. You need a remote service, that returns a simple JSON that looks like
   ```json
   {
     "rateLimitForTiers": {
       "tier1": "10/minute",
       "tier2": "20/minute",
       "tier3": "50/minute"
     }
   }
   ```
   
   It must have a toplevel field named `rateLimitForTiers`, and it can have
   other toplevel fields. Within `rateLimitForTiers` it should have at least the
   three fields {`tier1`, `tier2`, `tier3`}, and the value of each field should
   be a string of the form `"NUM/TIMEUNIT"` where the `NUM` is an integer and
   the timeunit is one of `minute`, `hour`, `day`.

   The service can vary the values over time, but the structure should remain the same.

4. You need the usual for VerifyAPIKey:
   - a Developer
   - an API Product, that authorizes at least `GET /t1` on THIS proxy
   - an App, that grants credentials to the developer on that API product.

   If you're not clear on how to set that up, [this screencast](https://www.youtube.com/watch?v=4FYZQuKuG3Y) will help.


5. Also create a 2nd app for the same product. It can be for the same developer if you like.   There will be a different credential.

6. For each app, add a custom attribute named "tier", and set the value of that
   custom attribute to be one of {`tier1`, `tier2`, `tier3`}.

7. Copy the Consumer Key for each of the apps, into shell variables.
   Call them `CLIENTID1` and `CLIENTID2` if you like.


## Using it

Run this command repeatedly to see the quota enforcement:
```sh
apigee=https://my-apigee-endpoint.io
curl -i -H APIKEY:$CLIENTID1 $apigee/lookup-quota-externally/t1
```

If you like you can use an infinite loop in your shell:
```sh
while true ; do ; curl -q -i -H APIKEY:$CLIENTID1 $apigee/lookup-quota-externally/t1 ; printf "\n\n"; sleep 1 ; done
```

If you do that, you should see the quota being "consumed" by each call, until
the proxy begins returning 429s. Then, after the quota resets, calls will again
be allowed through.

Headers in each response will show you the status of the quota:
```
HTTP/2 200
content-type: application/json
apiproxy: lookup-quota-externally r14
ratelimit: limit=20, remaining=19, reset=49
quota-reset: 2024-02-08T22:24:00Z
x-request-id: f00073fe-0f03-4a1f-ae31-e44d9c6b226f
content-length: 40
date: Thu, 08 Feb 2024 22:23:11 GMT
via: 1.1 google, 1.1 google
alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000

{
  "status": "ok",
  "tier": "tier2"
}
```

Specifically, the `ratelimit` header and the `quota-reset` header.


## Teardown

Remove the apps, the product, the developer, and the proxy from your Apigee. 


## Support

This example is open-source software, and is not a supported part of Apigee.  If
you need assistance, you can try inquiring on [the Google Cloud Community forum
dedicated to Apigee](https://goo.gle/apigee-community) There is no service-level
guarantee for responses to inquiries posted to that site.


## License

This material is [Copyright 2024 Google LLC](./NOTICE).
and is licensed under the [Apache 2.0 License](LICENSE).


## Author

dchiesa@google.com
