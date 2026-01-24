# Nextcloud Buildpack for Scalingo

Buildpack pour télécharger, installer et configurer Nextcloud sur Scalingo PaaS.

## Installation

Dans votre fichier `.buildpacks` :
```
https://github.com/Scalingo/apt-buildpack.git
https://github.com/Beneylu/scalingo-nextcloud-buildpack.git
https://github.com/Scalingo/php-buildpack.git
```

## Variables d'environnement

### Requises

| Variable | Description | Exemple |
|----------|-------------|---------|
| `NEXTCLOUD_VERSION` | Version de Nextcloud | `32.0.5` |
| `NEXTCLOUD_TRUSTED_DOMAIN` | Domaine autorisé | `xxx.scalingo.io` |
| `NEXTCLOUD_ADMIN_USER` | Utilisateur admin | `admin` |
| `NEXTCLOUD_ADMIN_PASSWORD` | Mot de passe admin | `<password>` |

### Stockage S3 (Scaleway)

| Variable | Description |
|----------|-------------|
| `SCALEWAY_S3_BUCKET` | Nom du bucket |
| `SCALEWAY_ACCESS_KEY` | Clé d'accès |
| `SCALEWAY_SECRET_KEY` | Clé secrète |
| `SCALEWAY_S3_REGION` | Région (défaut: `fr-par`) |
| `SCALEWAY_S3_HOSTNAME` | Hostname (défaut: `s3.fr-par.scw.cloud`) |

### Pour les redéploiements

Après la première installation, sauvegarder et configurer :
- `NEXTCLOUD_INSTANCEID`
- `NEXTCLOUD_PASSWORDSALT`
- `NEXTCLOUD_SECRET`

### Auto-fournies par Scalingo
- `SCALINGO_POSTGRESQL_URL` - Base de données

## Ce que fait le buildpack

1. **Télécharge** Nextcloud depuis le site officiel
2. **Extrait** et installe dans `/app`
3. **Pré-installe** l'application OnlyOffice
4. **Configure** nginx pour Nextcloud
5. **Bootstrap** automatique au démarrage :
   - Détection fresh install vs redéploiement
   - Configuration S3 automatique
   - Création des tables si nécessaire

## Structure

```
bin/
  compile    # Script principal d'installation
  detect     # Détection du buildpack
  release    # Configuration de release
```

## Statut

**FONCTIONNEL** - Déployé en production sur Scalingo SecNumCloud.
