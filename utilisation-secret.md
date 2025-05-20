

---

## 🔐 Objectif :

* Stocker `cosign.key` et `cosign.pub` dans un `Secret` OpenShift
* Monter ce secret dans un pod ou un script
* Utiliser ces clés pour signer et vérifier une image (ex : dans un pipeline CI/CD ou script local)

---

## ✅ 1. Générer les clés (si pas encore fait)

Sur ta machine Windows :

```powershell
cosign generate-key-pair
```

Tu auras deux fichiers :

* `cosign.key` (privée)
* `cosign.pub` (publique)

---

## ✅ 2. Créer un `Secret` OpenShift avec les clés

```bash
oc create secret generic cosign-keys \
  --from-file=cosign.key=cosign.key \
  --from-file=cosign.pub=cosign.pub
```

> Tu peux vérifier que le secret existe :

```bash
oc get secret cosign-keys
```

---

## ✅ 3. Utiliser ces clés pour signer une image

### a. Extraire les clés temporairement depuis le `Secret`

Tu peux le faire dans un pod, un CI/CD pipeline, ou localement via `oc` :

```bash
# Extraire la clé privée depuis le secret
oc extract secret/cosign-keys --to=./cosign-keys --confirm
```

Cela va créer localement :

```bash
cosign-keys/
├── cosign.key
└── cosign.pub
```

### b. Signer l’image :

```bash
cosign sign --key cosign-keys/cosign.key \
  artifactory.gc.com/my-project/springboot-app:latest
```

---

## ✅ 4. Vérifier une image avec la clé publique

```bash
cosign verify --key cosign-keys/cosign.pub \
  artifactory.gc.com/my-project/springboot-app:latest
```

---

## ✅ 5. Bonus : Utilisation dans un Pod (pipeline Tekton ou Job)

Voici un exemple de **montage du secret cosign dans un Pod** :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: signer
spec:
  containers:
  - name: signer
    image: your-image-with-cosign
    command: ["sleep", "3600"]
    volumeMounts:
    - name: cosign-keys
      mountPath: "/cosign"
  volumes:
  - name: cosign-keys
    secret:
      secretName: cosign-keys
```

Et dans ce Pod :

```bash
cosign sign --key /cosign/cosign.key your-image
```

---

## 🧼 Nettoyage (facultatif)

Si tu veux supprimer temporairement les fichiers extraits localement :

```bash
rm -rf ./cosign-keys
```

---
