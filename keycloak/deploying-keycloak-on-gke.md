# Deploying production-ready keycloakx to GKE and exposing it with nginx ingress controller
I visited many guides online, although many gave me a good base to understand how keycloak works, none was 100% up-to-date and 100% compatible with my requirements. This is what worked for me.

### Requirements/setup
* Must be deployed to a regular GKE cluster
* Must use [keycloakx](https://github.com/codecentric/helm-charts/tree/master/charts/keycloakx), not [keycloak](https://github.com/codecentric/helm-charts/tree/master/charts/keycloak) (deprecated).
* Must connect to an external Postgres DB running in CloudSQL
* Can only be accesed with HTTPS.
* Nginx ingress controller already installed - e.g https://www.pluralsight.com/cloud-guru/labs/gcp/configuring-the-nginx-ingress-controller-on-gke
* TLS secret already present - e.g https://kubernetes.io/docs/reference/kubectl/generated/kubectl_create/kubectl_create_secret_tls/

### values.yaml
```
dbchecker:
  enabled: true

database:
  vendor: <<postgres|mysql>>
  hostname: <<database-ip-or-dns>>
  port: <<5432|3306>>
  username: "keycloak"
  password: "<<yourpassword>>"
  database: keycloak

command:
  - "/opt/keycloak/bin/kc.sh"
  - "start"
  - "--http-port=8080"
  - "--spi-events-listener-jboss-logging-success-level=info"
  - "--spi-events-listener-jboss-logging-error-level=warn"
  - "--verbose"

extraEnv: |
  - name:  KC_HOSTNAME_STRICT
    value: "false"
  - name:  KC_HTTP_RELATIVE_PATH # This is necessary and safe for using a reverse proxy/ingress/load balance.
    value: "/auth"
  - name:  KC_HOSTNAME
    value: "<<your-url.com>>"
  - name:  KC_HEALTH_ENABLED # Healthcheck
    value: "true"
  - name:  KC_HTTP_ENABLED # This is necessary and safe for using a reverse proxy/ingress/load balance.
    value: "true"
  - name:  KC_PROXY_HEADERS # This is necessary and safe for using a reverse proxy/ingress/load balance.
    value: "xforwarded"
  - name: KEYCLOAK_ADMIN # Admin username
    value: <<admin>>
  - name: KEYCLOAK_ADMIN_PASSWORD # Configured down below
    value: '{{.Values.adminUserPassword}}'
  - name: JAVA_OPTS_APPEND
    value: >-
      -XX:+UseContainerSupport
      -XX:MaxRAMPercentage=50.0
      -Djava.awt.headless=true
      -Djgroups.dns.query={{ include "keycloak.fullname" . }}-headless


adminUserPassword: <<adminuserpassword>>
metrics:
  enabled: true
service:
  type: LoadBalancer
  httpPort: 8080

ingress:
  enabled: true
  ingressClassName: "nginx"
  servicePort: http
  rules:
    -
      host: '<<your-url.com>>'
      paths:
        - path: '{{ tpl .Values.http.relativePath $ | trimSuffix "/" }}/'
          pathType: Prefix
  tls:
    - hosts:
        - "<<your-url.com>>"
      secretName: "<<tls-secret-name>>"
```
### Deployment commands
1. helm repo add codecentric https://codecentric.github.io/helm-charts
2. helm repo update
3. helm upgrade --install keycloak -nkeycloak codecentric/keycloakx -f values.yaml

4. Pick up the ingress public IP with `kubectl get ingress -nkeycloak` and add it to your DNS registry
5. If this helped you, let me know at https://www.linkedin.com/in/lima-michel/
---


### Helpful resources that helped me
https://www.keycloak.org/server/all-config
https://www.keycloak.org/server/reverseproxy
https://blog.devops.dev/deploy-keycloak-v24-to-k8s-cluster-with-helm-83e6714f2888
https://groups.google.com/g/keycloak-user/c/A8CSb9G-b34
