# appsody-security-sample

The Open Liberty Appsody stack provides default security credentials for local development. These default values (in configDropins/default/quick-start-security.xml) are removed from the official deployment image for security purposes with the intent that a user will add their own security mechanism when deploying. This applications shows a very basic way in which that can be accomplished using environment variables and an injected Secret. Additionally, a Service Monitor configuration allows for easy connection from a Prometheus instance.

## quick-start-security

As you can see in the app, the quick-start-security configuration is moved to server.xml where it will not be removed. It also uses environment variables as values for username and password.

```xml
<quickStartSecurity userName="${stack.username}" userPassword="${stack.password}" />
```

## Secret Injection

To set the username and password environment variables, the app is injecting values from a pre-defined Secret called "mySecret". This must exist in the cluster namespace before deploying.

```yaml
env:
  - name: STACK_USERNAME
  valueFrom:
    secretKeyRef:
      key: username
      name: mySecret
  - name: STACK_PASSWORD
  valueFrom:
    secretKeyRef:
      key: password
      name: mySecret
```

## HTTPS Port Configuration

You will notice that the app-deploy.yaml is configured to use port 9443 rather than port 9080.

```yaml
port: 9443
```

## Service Monitor

To allow for easy detection from Prometheus, this app defines a Service Monitor that will use the credentials from "mySecret" and the HTTPS port to connect to the deployment's /metrics endpoint.

```yaml
monitoring:
  endpoints:
  - basicAuth:
      password:
        key: password
        name: mysecret
      username:
        key: username
        name: mysecret
    interval: 5s
    port: HTTPS
    scheme: HTTPS
    tlsConfig:
      insecureSkipVerify: true
  labels:
    app-monitoring: ''
```

Prometheus may need to be configured to watch the namespace where this app is deployed, and the namespace will need to have the label "app-monitoring."
