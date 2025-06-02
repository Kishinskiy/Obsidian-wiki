
1. Create a cert of your custom gitlab host. I did it with
	```sh
openssl s_client -showcerts -connect {your.gitlab.hostname}:443 -servername {your.gitlab.hostname} < /dev/null 2>/dev/null | openssl x509 -outform PEM > {your.gitlab.hostname}.crt
	```
2. Then create a secret in your k8s-cluster in the namespace where gitlab-runner has been deployed
   ```sh
   kubectl create secret generic gitlab-cert-name \
  --namespace <NAMESPACE> \
  --from-file={your.gitlab.hostname}.crt
```
3. In your helm-chart values.yaml set
   ```sh
   certsSecretName: gitlab-cert-name
```
