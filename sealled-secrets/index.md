# Managing multi-cluster secrets with Sealed Secrets

One of the most hard problems i have found dealing with multiple-cluster in my jobs is dealing with secrets management, create secrets for each cluster, being sure that secret works, and store them safely.

Having a proper control of this issue always be a big mess, mostly because this easily become a safety issue 
most of the time in companies i have passed, someone have all the secrets stored in their machine or in some dark and scared place, or all of the secrets start to be stored in one private repository where every one in a point of the time will have the access or other horrible solution like that.

In my researches about developer experience i have found some people talking about one open-source project called <b>Sealed Secrets</b> and in the moment i looked for it, i instantly fall in love.

The concept is really simple and powerful i an application focused on kubernetes, which receive some data and encrypt it and transform into an k8s custom resource.

At first you may think "why do i need this, i can only place my secret in my cluster right?"
well yes, but when we start to create custom pipelines to improve our deployment process, create multiple clusters, or increase a lot our internal applications, store this secrets and apply them again become a real problem.

With sealed secrets we can create the secret based on a specific certificate, and store them anywhere we want, and in the moment we apply that sealed-secret the service will generate the real secret.


## Installing

### Installing sealed secrets
```
kubectl create namespace sealed-secrets
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm install sealed-secrets sealed-secrets/sealed-secrets --namespace sealed-secrets --version 2.1.8
```

### Installing cli
Enter on [Release URL](https://github.com/bitnami-labs/sealed-secrets/releases/tag/v0.17.5) and download the version for your machine, in my case will be linux-amd64, here we will do this using wget.
```
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.17.5/kubeseal-0.17.5-linux-amd64.tar.gz
tar -xzf kubeseal-0.17.5-linux-amd64.tar.gz
sudo mv kubeseal /usr/local/bin/
```

## How To

### Creating a  new secret

#### Key Value Secret
```
kubectl create secret generic <YOUR_SECRET> --dry-run=client --from-literal=<KEY>=<VALUE> -o yaml | \
    kubeseal \
      --controller-name=sealed-secrets \
      --controller-namespace=sealed-secrets \
      --format yaml > <SEALED_SECRET_FILE>.yaml
```

#### File Secret
```
kubectl create secret generic <YOUR_SECRET> --dry-run=client --from-file=<FILE> -o yaml | \
    kubeseal \
      --controller-name=sealed-secrets \
      --controller-namespace=sealed-secrets \
      --format yaml > <SEALED_SECRET_FILE>.yaml
```

### Applying sealed secret
```
kubectl create -f <SEALED_SECRET_FILE>.yaml
```

## Certificate Management
The sealed secret controller with generate certificates when is deployed, and will generate new certificate at some period of time, but you can add your own certificates.

> :information_source: when a new certificate is added, the old ones isn't deleted, so old generated sealed secrets can be added again if you have the cert applied.

### adding a new cert
```
kubectl -n sealed-secrets create secret tls <CERT_NAME> --cert=<PUBLIC_KEY_FILE> --key=<PRIVATE_KEY_FILE>
kubectl -n sealed-secrets label secret <CERT_NAME> sealedsecrets.bitnami.com/sealed-secrets-key=active
```
after that restart the sealed secrets controller pod, and you are good to go, now you can get all the encrypted secrets you created and can put into a repository without fear.

This are the first steps to deal with sealed secrets, if you want to understand more about what Sealed Secrets really works i recommended you to follow the [Project Repository](https://github.com/bitnami-labs/sealed-secrets).

