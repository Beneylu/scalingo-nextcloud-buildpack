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

### user_oidc (optionnel)

| Variable | Description | Défaut |
|----------|-------------|--------|
| `USER_OIDC_APP_VERSION` | Version de l'app `user_oidc` à pré-installer | `8.8.0` |

## Ce que fait le buildpack

1. **Télécharge** Nextcloud depuis le site officiel
2. **Extrait** et installe dans `/app`
3. **Pré-installe** les applications OnlyOffice et `user_oidc`
4. **Configure** nginx pour Nextcloud
5. **Bootstrap** automatique au démarrage :
   - Détection fresh install vs redéploiement
   - Configuration S3 automatique
   - Création des tables si nécessaire
   - Activation et configuration de `user_oidc`

## Authentification OIDC

Le buildpack pré-installe l'application [`user_oidc`](https://github.com/nextcloud/user_oidc) et l'active automatiquement au démarrage.

Les réglages suivants sont appliqués à chaque démarrage :

| Clé | Valeur | Effet |
|-----|--------|-------|
| `enable_default_claims` | `false` | Désactive le mapping automatique des claims par défaut |
| `enrich_login_id_token_with_userinfo` | `true` | Enrichit le token avec les infos de l'endpoint userinfo |
| `send_userinfo_claims` | `false` | Ne retransmet pas les claims userinfo lors des requêtes sortantes |

> La configuration d'un provider OIDC (client ID, secret, discovery URL…) n'est pas encore automatisée.
> Elle doit être effectuée manuellement via l'interface d'administration ou `occ`.

## Structure

```
bin/
  compile    # Script principal d'installation
  detect     # Détection du buildpack
  release    # Configuration de release
```

## Statut

**FONCTIONNEL** - Déployé en production sur Scalingo SecNumCloud.
