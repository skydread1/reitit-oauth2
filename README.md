# Reitit-oauth2
---

## Rational
---

The commonly used library [weavejester/ring-oauth2](https://github.com/weavejester/ring-oauth2), providing a Ring middleware that acts as a OAuth 2.0 client, does not work with reitit.

The reason why it doesn't work with reitit is related to `sessions` and you can find the explanation of the problem in this [issue](https://github.com/metosin/reitit/issues/205).

## Goals
---

As for now, only `access-token` is handled (`refresh-token` is not handled yet).

The API function provides reitit routes for oauth2 (launch oauth2 and redirect) that you can merge with the rest of your app routes. the API function takes a map with the different services config (you can find an example of configs [here](https://github.com/skydread1/reitit-oauth2/blob/master/example/src/oauth_test/ring_handler.clj#L25)).

## Oauth2 setup steps (GOOGLE example)
---

### 1) Create project in google dev console

In the [google dev console](https://console.cloud.google.com/), create a project.

### 2) Oauth consent screen

In the `oauth consent screen` tab, fill the app info that is going to be displayed to the user upon giving permissions.

You can also select the permsissions you want the user to give to you.

### 3) Credentials

In the `credentials` tab, click `create crednetials` then `OAuth client ID` with type `Web application`.

For `Authorised JavaScript origins`, you need to specify you app URI. For local development, you need to add localhost as well and one entry per port.
For example, if you have a backend port 8123 and a fighweel front-end port 9500 you will add 2 URIs.

For `Authorised redirect URIs`, same remarks, one callback per port.

Here is an example:

![image](https://user-images.githubusercontent.com/16139969/200980756-a28072dc-6f93-4be0-9ea9-083326ca68f5.png)

Once you save, you should get your `client-id` and `client-secret`.

### 4) Save credentials and create config.

We then advice to store the configs in a edn file that you will slurp in your code or env variables and of course never pushing the credentials to your online repo (at least the `client-secret`).

For our google example, our google config edn file look like this

```clojure
{:google {:project-id       "my-website"
          :authorize-uri    "https://accounts.google.com/o/oauth2/auth"
          :access-token-uri "https://oauth2.googleapis.com/token"
          :client-id        "CLIENT-ID"
          :client-secret    "CLIENT-SECRET"
          :scopes           ["https://www.googleapis.com/auth/userinfo.email"
                             "https://www.googleapis.com/auth/userinfo.profile"]
          :launch-uri       "/oauth/google/login"
          :redirect-uri     "http://localhost:8123/oauth/google/callback" ;; would need be sure to change the port depending if you need to.
          :landing-uri      "/oauth/google/success"}}
```

## Implementation Example
---

A good example written by the author of the upstream library can be found alongside the source in this repo to get you started.

## Caveats
---

### wrap-session

In order to make the session works, you must follow the workaround highlighted in this [issue](https://github.com/metosin/reitit/issues/205)

### wrap-params and middleware orders

- `wrap-params` must be in the middleware stack (or any middleware adding `params` and `body-params` to the request such as `muuntaja/format-middleware` for instance).

- Be aware of the order of your middlewares, for more details, see this [issue](https://github.com/weavejester/ring-oauth2/issues/43).

