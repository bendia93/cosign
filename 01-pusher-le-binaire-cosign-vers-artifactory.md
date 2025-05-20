## ✅ Objectif :

* Télécharger le binaire officiel `cosign`
* L’uploader dans **un repository `generic` ou `docker`** de ton Artifactory
* Utiliser ce binaire depuis des scripts, containers ou CI/CD internes

---

## 📦 1. Télécharger le binaire `cosign`

### Pour Linux (ex. pour l’usage dans conteneurs) :

```bash
curl -LO https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64
chmod +x cosign-linux-amd64
mv cosign-linux-amd64 cosign
```

### Pour Windows (si besoin) :

```powershell
Invoke-WebRequest -Uri "https://github.com/sigstore/cosign/releases/latest/download/cosign-windows-amd64.exe" -OutFile "cosign.exe"
```

---

## 🗃️ 2. Préparer l’upload vers Artifactory

Tu dois avoir :

* Un **repository générique** dans Artifactory (ex. `tools-generic-local`)
* Des **identifiants** valides (ou token/API key)

### Exemple de dépôt :

```
https://artifactory.gc.com/artifactory/tools-generic-local/cosign/cosign-linux-amd64
```

---

## ☁️ 3. Uploader avec `curl` (méthode simple)

### a. Commande curl :

```bash
curl -u "<USER>:<PASSWORD_OR_API_KEY>" \
  -T cosign \
  "https://artifactory.gc.com/artifactory/tools-generic-local/cosign/cosign-linux-amd64"
```

> 🔐 Remplace `USER` et `PASSWORD_OR_API_KEY` par tes identifiants ou un token

### b. Résultat :

Tu pourras ensuite accéder à `cosign` depuis n’importe où dans ton réseau privé via :

```bash
https://artifactory.gc.com/artifactory/tools-generic-local/cosign/cosign-linux-amd64
```

---

## 🤖 4. Utilisation dans un script ou un Dockerfile

### Exemple dans un Dockerfile :

```Dockerfile
RUN curl -u "$USER:$TOKEN" -LO \
  https://artifactory.gc.com/artifactory/tools-generic-local/cosign/cosign-linux-amd64 \
  && chmod +x cosign-linux-amd64 \
  && mv cosign-linux-amd64 /usr/local/bin/cosign
```

> Tu peux injecter `USER` et `TOKEN` via des `ARG` ou `secrets` Docker ou OpenShift

---

## ✅ Bonnes pratiques

* Stocke **chaque version** dans un sous-répertoire, par exemple :
  `.../cosign/v2.2.2/cosign-linux-amd64`
* Tu peux aussi stocker le **hash SHA256** dans un fichier `.sha256` à côté
* Protège le dépôt avec des permissions `read-only` pour les consommateurs

