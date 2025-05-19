
## ✅ Résumé du flux :

1. Construire l’image avec `oc new-build`
2. Récupérer l’image depuis le registre interne d’OpenShift
3. Tagger et pousser vers `artifactory.gc.com`
4. Signer avec `cosign` (en local)
5. (Optionnel) Redéployer depuis `artifactory.gc.com` via OpenShift

---

## ✅ 1. Build de l'image dans OpenShift

```bash
# Crée le build S2I Java 21
oc new-build --name=springboot-app \
  --binary=true \
  --image-stream=java:21 \
  --strategy=source

# Lance la build avec ton code source local
oc start-build springboot-app --from-dir=. --follow
```

L’image est construite dans :

```text
image-registry.openshift-image-registry.svc:5000/<namespace>/springboot-app:latest
```

---

## ✅ 2. Pull l’image depuis le registre interne

### a. Se connecter au registre interne avec podman/docker :

```bash
podman login \
  -u $(oc whoami) \
  -p $(oc whoami -t) \
  image-registry.openshift-image-registry.svc:5000
```

### b. Pull l’image :

```bash
podman pull image-registry.openshift-image-registry.svc:5000/<namespace>/springboot-app:latest
```

### c. Tag pour Artifactory :

```bash
podman tag \
  image-registry.openshift-image-registry.svc:5000/<namespace>/springboot-app:latest \
  artifactory.gc.com/my-project/springboot-app:latest
```

---

## ✅ 3. Push vers `artifactory.gc.com`

Tu as déjà un secret dans OpenShift, donc tu peux t’en servir avec `podman login` :

```bash
# Se connecter à Artifactory
podman login artifactory.gc.com
```

> Utilise les identifiants du secret (ou extrais-les via `oc get secret` si nécessaire)

### Puis push :

```bash
podman push artifactory.gc.com/my-project/springboot-app:latest
```

---

## ✅ 4. Signer avec `cosign`

```bash
# Générer une clé si pas encore fait
cosign generate-key-pair

# Signer
cosign sign --key cosign.key artifactory.gc.com/my-project/springboot-app:latest
```

---

## ✅ 5. (Optionnel) Déployer depuis Artifactory dans OpenShift

Tu peux maintenant créer un déploiement qui **pull directement depuis Artifactory**, en utilisant ton secret existant :

### Exemple `deployment.yaml` :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: springboot-app
  template:
    metadata:
      labels:
        app: springboot-app
    spec:
      containers:
        - name: springboot-app
          image: artifactory.gc.com/my-project/springboot-app:latest
          ports:
            - containerPort: 8080
      imagePullSecrets:
        - name: artifactory-secret
```

---

Souhaites-tu un **script Bash complet** pour automatiser tout ça (build → pull → push → sign) ?
