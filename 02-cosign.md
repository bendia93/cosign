# Install 
https://github.com/sigstore/cosign/releases/tag/v2.5.0

 

---

## ✅ 1. Installer `cosign` sur Windows

### Option 1 : via `scoop` (recommandé si tu l’as)

```powershell
scoop install cosign
```

### Option 2 : via GitHub Releases (manuel)

1. Va sur la page : [https://github.com/sigstore/cosign/releases](https://github.com/sigstore/cosign/releases)
2. Télécharge le fichier `.exe` correspondant à Windows :
   Exemple : `cosign-windows-amd64.exe`
3. Renomme-le en `cosign.exe` et place-le dans un dossier comme `C:\Tools\Cosign`
4. Ajoute ce dossier à ta variable d’environnement **`PATH`** :

   * Panneau de configuration → Système → Variables d’environnement → Path → Ajouter

---

## ✅ 2. Générer une paire de clés pour signer

Dans PowerShell ou un terminal :

```powershell
cosign generate-key-pair
```

Cela va créer deux fichiers :

* `cosign.key` : clé privée (🚫 ne la partage jamais)
* `cosign.pub` : clé publique (à stocker dans Git ou partager pour la vérification)

Tu peux aussi utiliser une phrase de passe facultative (tu peux appuyer sur `Entrée` pour ne pas en mettre).

---

## ✅ 3. Signer une image Docker

### Supposons que tu aies déjà **poussé une image vers Artifactory**, par exemple :

```
artifactory.gc.com/my-project/springboot-app:latest
```

### Commande pour signer :

```powershell
cosign sign --key cosign.key artifactory.gc.com/my-project/springboot-app:latest
```

> 🔐 `cosign` se connectera automatiquement à ton daemon Docker (`docker` ou `podman`) si nécessaire.

---

## ✅ 4. Vérifier la signature

Tu peux vérifier la signature avec :

```powershell
cosign verify --key cosign.pub artifactory.gc.com/my-project/springboot-app:latest
```

---

## ✅ 5. Cas de registre privé (comme Artifactory)

Si ton registry Artifactory nécessite une authentification (ce qui est presque toujours le cas), tu dois :

### a. Te connecter d’abord :

```powershell
docker login artifactory.gc.com
```

ou

```powershell
podman login artifactory.gc.com
```

### b. Puis signer avec authentification (facultatif, `cosign` utilise les credentials Docker par défaut) :

```powershell
cosign sign --key cosign.key \
  --registry-username your-user \
  --registry-password your-pass \
  artifactory.gc.com/my-project/springboot-app:latest
```

---

## 🔐 Recommandations sécurité

* Ne mets **jamais `cosign.key` dans Git** ni dans un répertoire partagé.
* Tu peux stocker la clé dans un coffre-fort (Azure Key Vault, HashiCorp Vault, etc.).
* Tu peux aussi utiliser **`cosign` avec des clés de sécurité hardware (YubiKey, TPM, Azure KMS, etc.)** si besoin.

---

