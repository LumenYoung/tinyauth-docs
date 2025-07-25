---
sidebar_position: 3
---

# Headers

Setting headers can be useful for authenticating to apps with the credentials from Tinyauth. While Tinyauth offers some defaults, it also allows you to set any headers you like that will be automatically returned in the authentication server response. This could be useful when your application supports header based authentication, where the app trusts the reverse proxy to provide the authentication and use the user information passed from the header.

Note that the headers mentioned here is not automatically sent to the apps unless you specify it on the reverse proxy middleware you are using. See the section below for how to specify passing header in your reverse proxy.

## Supported headers

### Remote user

The `Remote-User` header contains the username of the currently logged in user.

If you are using an OAuth provider, Tinyauth will try to retrieve the `preferred_username` claim from the OIDC response. If it isn't included in the response, Tinyauth will make a pseudo one using your email address in the format of `username_domain.com`.

:::info
Headers are case-insensitive. Therefore you can use either `Remote-User` or `remote-user` for specifying the header.
:::

### Remote email

The `Remote-Email` header contains the email of the currently logged in user.

If you are using simple username and password authentication, a pseudo email address will be created using your username and the currently configured domain. If you are using OAuth then the email from your OAuth provider will be used (retrieved from the `email` claim).

### Remote name

The `Remote-Name` header contains the full name of the currently logged in user.

If you are using simple username and password authentication or your OAuth provider does not provide the `name` claim, a pseudo name will be created either with either your username in the format of `User` or with your email in the format of `User (example.com)`.

### Remote groups

The `Remote-Groups` header contains the groups of the currently logged in user. They are retrieved from the `groups` claim in your OIDC server. They can be used to allow access to specific user groups configured by your OIDC server. For more information check the [OIDC access controls](/docs/guides/access-controls.md#access-controls-using-oidc-groups) guide.

### Custom headers

You can set the `tinyauth.headers` label on any container that uses the Tinyauth middleware and it will automatically add them to its response. For example, you can have the following line in your app's labels:

```yaml
tinyauth.headers: my-header=cool
```

And when you authenticate to your app through Tinyauth, your app will receive the `my-header` header.

:::warning
Make sure to create a list of trusted proxy URLs that your app accepts headers from. If your app trusts all proxies then anyone can just send the header to your app and possibly bypass any authentication you have set.
:::

:::info
By default Tinyauth will use the subdomain name of the request and it will try to find a container with a matching name in order to search for the labels. For example, if the request host is `myapp.example.com`, Tinyauth will check for labels in the container named `myapp`. You can change this behavior using the `tinyauth.domain` label. For more information check the [access controls](../guides/access-controls.md#label-discovery) guide.
:::

## Adding headers to proxy

To add the headers to the proxy responses you need to configure your proxy to forward the headers. This varies from proxy to proxy.

### Traefik

Just add the following in the Tinyauth container lables:

```yaml
traefik.http.middlewares.tinyauth.forwardauth.authResponseHeaders: remote-user # This can be a comma separated list of more headers you will like to copy like the custom ones you set
```

### Caddy

Just add the following label in the Caddy labels:

```yaml
caddy.forward_auth.copy_headers: remote-user
```

Multiple headers here are separated by space. Therefore multiple headers are passed like `remote-user remote-name remote-email remote-groups`

### Nginx/Nginx Proxy Manager

Add the following lines after the `error_page 401 = @tinyauth_login;`:

```shell
auth_request_set $tinyauth_remote_user $upstream_http_remote_user;
proxy_set_header remote-user $tinyauth_remote_user;
```

You can repeat this step multiple times to add more headers, for example:

```shell
auth_request_set $my_header $upstream_http_my_header;
proxy_set_header my-header $my_header;
```
