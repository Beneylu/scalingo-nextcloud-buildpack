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

### Provisioning automatique du provider OIDC

Si les quatre variables obligatoires ci-dessous sont définies, le provider OIDC est automatiquement créé ou mis à jour à chaque démarrage (idempotent).

**Variables obligatoires** (si absentes, le provisioning est simplement ignoré) :

| Variable | Description |
|----------|-------------|
| `OIDC_PROVIDER_IDENTIFIER` | Identifiant du provider dans Nextcloud |
| `OIDC_CLIENT_ID` | Client ID OAuth2 |
| `OIDC_CLIENT_SECRET` | Client secret OAuth2 |
| `OIDC_DISCOVERY_URI` | URL du document de découverte OIDC |

**Variables optionnelles** :

| Variable | Description | Défaut |
|----------|-------------|--------|
| `OIDC_SCOPE` | Scopes demandés | `openid` |
| `OIDC_MAPPING_UID` | Claim utilisé comme identifiant Nextcloud | `sub` |
| `OIDC_EXTRA_CLAIMS` | Claims supplémentaires à transmettre | _(vide)_ |
| `OIDC_UNIQUE_UID` | UID unique entre providers | `1` |
| `OIDC_CHECK_BEARER` | Vérifier le Bearer token | `0` |
| `OIDC_SEND_ID_TOKEN_HINT` | Envoyer l'ID token hint à la déconnexion | `0` |

**Exemple de configuration** :

```
OIDC_PROVIDER_IDENTIFIER=example
OIDC_CLIENT_ID=your-client-id
OIDC_CLIENT_SECRET=your-client-secret
OIDC_DISCOVERY_URI=https://example.com/.well-known/openid-configuration
OIDC_SCOPE=openid
OIDC_MAPPING_UID=sub
```

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

## Journaux

- **`log_type`** : `file` — fichier **`/tmp/nextcloud-data/nextcloud.log`** (même répertoire que le datadir local du buildpack).
- **`loglevel`** : `0` (debug) pour le diagnostic et la **Journalisation** ; repasse à `2`–`3` en production pour limiter le bruit.
- Au **démarrage du conteneur**, le datadir reçoit `chmod 0777` et le fichier journal `0666` pour que **PHP-FPM** (pas le script bootstrap) puisse écrire ; sans cela, Nextcloud retombe sur `error_log` et l’UI **Journalisation** reste vide.
- À chaque boot, si l’instance est déjà installée, les clés **`log_type` / `logfile` / `loglevel`** sont resynchronisées via `occ` (anciennes instances encore en `errorlog` sont corrigées).
- Installe l’app **Log Reader** (`logreader`) depuis le store si tu utilises cette interface plutôt que la page native **Administration → Journalisation**.

## Authentification OIDC

Le buildpack pré-installe l'application [`user_oidc`](https://github.com/nextcloud/user_oidc) et l'active automatiquement au démarrage.

Ces réglages sont écrits dans `config.php` sous `$CONFIG['user_oidc']` (reconstruction au redéploiement ; fusion après `maintenance:install` pour une nouvelle instance) — c’est ce que lit `user_oidc` via la config système :

| Clé | Valeur | Effet |
|-----|--------|-------|
| `enable_default_claims` | `false` | Désactive le mapping automatique des claims par défaut |
| `enrich_login_id_token_with_userinfo` | `true` | Enrichit le token avec les infos de l'endpoint userinfo |
| `send_userinfo_claims` | `false` | Ne retransmet pas les claims userinfo lors des requêtes sortantes |

> Si les variables `OIDC_PROVIDER_IDENTIFIER`, `OIDC_CLIENT_ID`, `OIDC_CLIENT_SECRET` et `OIDC_DISCOVERY_URI` sont définies,
> le provider est provisionné automatiquement à chaque démarrage via `occ user_oidc:provider` (idempotent).
> Voir la section [Provisioning automatique du provider OIDC](#provisioning-automatique-du-provider-oidc) ci-dessus.

## Structure

```
bin/
  compile    # Script principal d'installation
  detect     # Détection du buildpack
  release    # Configuration de release
```

## Statut

**FONCTIONNEL** - Déployé en production sur Scalingo SecNumCloud.
