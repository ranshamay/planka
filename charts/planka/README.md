# Planka Helm Chart

[Planka](https://github.com/plankanban/planka) is an OSS alternative to Trello that you can host yourself, and this is a Helm Chart to make it easier to deploy to K8s.

Shoutout to [this issue](https://github.com/plankanban/planka/issues/192) who have been asking for this Helm Chart.

## Issues

By using the Bitnami chart for PostgreSQL, there is an issue where once deployed, if trying to use a different password then it will be ignored as the Persistant Volume (PV) will already exist with the previous password. See warning from Bitnami below:

> **Warning!** Setting a password will be ignored on new installation in the case when previous Posgresql release was deleted through the helm command. In that case, old PVC will have an old password, and setting it through helm won't take effect. Deleting persistent volumes (PVs) will solve the issue. Refer to [issue 2061](https://github.com/bitnami/charts/issues/2061) for more details

If you want to fully uninstall this chart including the data, follow [these steps](https://github.com/bitnami/charts/blob/main/bitnami/postgresql/README.md#uninstalling-the-chart) from the Bitnami Chart's docs.

## Usage

The basic usage of the chart can be found below:

```bash
git clone https://github.com/Chris-Greaves/planka-helm-chart.git
cd planka-helm-chart
helm dependency build
export SECRETKEY=$(openssl rand -hex 64)
helm install planka . --set secretkey=$SECRETKEY
```

> **NOTE:** The command `openssl rand -hex 64` is needed to create a random hexadecimal key for planka. On Windows you can use Git Bash to run that command.

To access Planka you can port forward using the following command:

```bash
kubectl port-forward $POD_NAME 3000:3000
```

### Accessing Externally

To access Planka externally you can use the following configuration

```bash
# HTTP only
helm install planka . --set secretkey=$SECRETKEY \
--set ingress.enabled=true \
--set ingress.hosts[0].host=planka.example.dev \

# HTTPS
helm install planka . --set secretkey=$SECRETKEY \
--set ingress.enabled=true \
--set ingress.hosts[0].host=planka.example.dev \
--set ingress.tls[0].secretName=planka-tls \
--set ingress.tls[0].hosts[0]=planka.example.dev \
```

or create a values.yaml file like:

```yaml
secretkey: "<InsertSecretKey>"
ingress:
  enabled: true
  hosts:
    - host: planka.example.dev
      paths:
        - path: /
          pathType: ImplementationSpecific

# Needed for HTTPS
  tls:
   - secretName: planka-tls # existing TLS secret in k8s
     hosts:
       - planka.example.dev
```

```bash
helm install planka . -f values.yaml
```

### Things to consider if production hosting

If you want to host Planka for more than just playing around with, you might want to do the following things:

- Create a `values.yaml` with your config, as this will make applying upgrades much easier in the future.
- Create your `secretkey` once and store it either in a secure vault, or in your `values.yaml` file so it will be the same for upgrading in the future.
- Specify a password for `postgresql.auth.password` as there have been issues with the postgresql chart generating new passwords locking you out of the data you've already stored. (see [this issue](https://github.com/bitnami/charts/issues/2061))

Any questions or concerns, [raise an issue](https://github.com/Chris-Greaves/planka-helm-chart/issues/new).
