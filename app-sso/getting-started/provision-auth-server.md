# Provision an AuthServer

---

👉 This article assumes AppSSO is installed on your TAP cluster and that your TAP installation is correctly configured.
If you are unsure about your TAP installation, refer
to [its documentation](https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/index.html).
To install AppSSO, refer to the instructions in [Install AppSSO](../platform-operators/installation.md).

👉 AppSSO is installed automatically installed with the `run`, `iterate`, and `full` TAP profiles, no extra steps
required.

👉 To make sure AppSSO is installed on your cluster, you can run:

```shell
tanzu package installed list -A | grep "sso.apps.tanzu.vmware.com"
```

---

In this tutorial, you are going to:

1. Set up your first authorization server, in the `default` namespace
2. Ensure it is running, so that users can log in

![Diagram of AppSSO's components, with AuthServer and Identity Providers highlighted](../../images/app-sso/authserver-tutorial.png)

## Provision an AuthServer

Deploy your first Authorization Server along with an `RSAKey` key for signing tokens.

```yaml
---
apiVersion: sso.apps.tanzu.vmware.com/v1alpha1
kind: AuthServer
metadata:
  name: my-authserver-example
  namespace: default
  labels:
    name: my-first-auth-server
    env: tutorial
  annotations:
    sso.apps.tanzu.vmware.com/allow-client-namespaces: "default"
    sso.apps.tanzu.vmware.com/allow-unsafe-issuer-uri: ""
    sso.apps.tanzu.vmware.com/allow-unsafe-identity-provider: ""
spec:
  replicas: 1
  tls:
    disabled: true
  identityProviders:
    - name: "internal"
      internalUnsafe:
        users:
          - username: "user"
            password: "password"
            email: "user@example.com"
            emailVerified: true
            roles:
              - "user"
  tokenSignature:
    signAndVerifyKeyRef:
      name: "authserver-signing-key"

---
apiVersion: secretgen.k14s.io/v1alpha1
kind: RSAKey
metadata:
  name: authserver-signing-key
  namespace: default
spec:
  secretTemplate:
    type: Opaque
    stringData:
      key.pem: $(privateKey)
      pub.pem: $(publicKey)
```

You can wait for the `AuthServer` to become ready with:

```shell
kubectl wait --for=condition=Ready authserver my-authserver-example
```

Alternatively, you can inspect your `AuthServer` like any other resource:

```shell
kubectl get authservers.sso.apps.tanzu.vmware.com --all-namespaces
```

and you should see:

```shell
NAMESPACE NAME                  REPLICAS ISSUER URI                                         CLIENTS STATUS
default   my-authserver-example 1        http://my-authserver-example.default.<your domain> 0       Ready
```

As you can see your `AuthServer` gets an issuer URI templated. This is its entrypoint. You can find an `AuthServer`'s
issuer URI in its status:

```shell
kubectl get authservers.sso.apps.tanzu.vmware.com my-authserver-example --output go-template='{{ .status.issuerURI }}'
```

Open your `AuthServer`'s issuer URI in your browser. You should see a login page. Log in using username = `user` and
password = `password`.

> ℹ️ If the issuer URIs domain is not yours, then the AppSSO package installation needs to be updated.
> See [installation](../platform-operators/installation.md)

---

✋ Note that if you are using TKGm or TKGs, which have customizable in-cluster communication CIDR ranges, there is a
[known issue](../known-issues/cidr-ranges.md) regarding AppSSO making requests to external identity providers with
`http` rather than `https`.

---

## 💡 *The AuthServer spec, in detail*

Here is a detailed explanation of the `AuthServer` you have applied in the above section. This is intended to give you
an overview of the different configuration values that were passed in. It is not intended to describe all the
ins-and-outs, but there are links to related docs in each section.

Feel free to skip ahead.

### Metadata

```yaml
metadata:
  labels:
    name: my-first-auth-server
    env: tutorial
  annotations:
    sso.apps.tanzu.vmware.com/allow-client-namespaces: "default"
    sso.apps.tanzu.vmware.com/allow-unsafe-issuer-uri: ""
    sso.apps.tanzu.vmware.com/allow-unsafe-identity-provider: ""
```

The `metadata.labels` uniquely identify the AuthServer. They are used as selectors by `ClientRegistrations`, to declare
from which authorization server a specific client obtains tokens from.

The `sso.apps.tanzu.vmware.com/allow-client-namespaces` annotation restricts the namespaces in which you can create
a `ClientRegistrations` targeting this authorization server. In this case, the authorization server will only pick up
client registrations in the `default` namespace.

The `sso.apps.tanzu.vmware.com/allow-unsafe-...` annotations enable "development mode" features, useful for testing.
Those should not be used for production-grade authorization servers.

Lean more about [Metadata](../service-operators/metadata.md).

### TLS & issuer URI

```yaml
spec:
  tls:
    disabled: true
```

The `tls` field configures how and if to obtain a certificate for an `AuthServer` as to secure its issuer URI. In this
case we have disabled it. As a result we will get an issuer URI which uses plain HTTP.

__Note:__ Plain HTTP access is for getting-started development
only! [Learn more about a production readiness with TLS](../service-operators/issuer-uri-and-tls.md)

### Token Signature

```yaml
---
apiVersion: sso.apps.tanzu.vmware.com/v1alpha1
kind: AuthServer
# ...
spec:
  tokenSignature:
    signAndVerifyKeyRef:
      name: "authserver-signing-key"
---
apiVersion: secretgen.k14s.io/v1alpha1
kind: RSAKey
metadata:
  name: authserver-signing-key
  namespace: default
spec:
  secretTemplate:
    type: Opaque
    stringData:
      key.pem: $(privateKey)
      pub.pem: $(publicKey)
```

The token signing key is the private RSA key used to sign ID Tokens,
using [JSON Web Signatures](https://datatracker.ietf.org/doc/html/rfc7515), and clients use the public key to verify the
provenance and integrity of the ID tokens. The public keys used for validating messages are published
as [JSON Web Keys](https://datatracker.ietf.org/doc/html/rfc7517) at `{.status.issuerURI}/oauth2/jwks`. When using the
port-forward declared in the section above, JWKs are available
at [http://localhost:7777/oauth2/jwks](http://localhost:7777/oauth2/jwks).

The `spec.tokenSignature.signAndVerifyKeyRef.name` references a secret containing PEM-encoded RSA keys, both `key.pem`
and `pub.pem`. In this specific example, we are
using [Secretgen-Controller](https://github.com/vmware-tanzu/carvel-secretgen-controller), a TAP dependency, to generate
the key for us.

Lean more about [Token Signature](../service-operators/token-signature.md).

### Identity providers

```yaml
spec:
  identityProviders:
    - name: "internal"
      internalUnsafe:
        users:
          - username: "user"
            password: "password"
            email: "user@example.com"
            roles:
              - "user"
```

AppSSO's authorization server delegate login and user management to external identity providers (IDP), such as Google,
Azure Active Directory, Okta, etc. See diagram at the top of this page.

In this example, we use an `internalUnsafe` identity provider. As the name implies, it is _not_ an external IDP, but
rather a list of hardcoded user/passwords. As the name also implies, this is not considered safe for production. Here,
we declared a user with username = `user`, and password = `password`. For production setups,
consider using OpenID Connect IDPs instead.

The `email` and `roles` fields are optional for internal users. However, they will be useful when we want to use SSO
with a client application later in this guide.