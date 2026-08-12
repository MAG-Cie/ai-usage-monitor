# Changelog

Toutes les modifications notables de ce projet sont documentées dans ce
fichier.

Le format suit [Keep a Changelog](https://keepachangelog.com/fr/1.0.0/),
et ce projet adhère au [Semantic Versioning](https://semver.org/lang/fr/).

## [Unreleased]

### Added

- **Vue Historique** : onglet dédié — dépense réelle du mois, moyenne
  journalière, projection, jour pic, **écart réel vs estimé** et séries
  journalières. 7 jours en Free, illimité en Pro.
- **Export CSV (Pro)** : export de l'historique de dépense (jour, provider,
  source, réel/estimé, coût, tokens) depuis l'onglet Historique.
- **Mode dégradé** : un provider dont toutes les sources échouent affiche
  « indisponible » (jamais un faux 0 €) ; marqueur ⚠ quand les données sont
  partielles.
- **Distribution** : cibles Windows installeur NSIS + zip portable, pipeline
  GitHub Actions (tag `v*` → build + tests → *release* brouillon), documents
  [NETWORK.md](./NETWORK.md), [SECURITY.md](./SECURITY.md) et licence
  propriétaire [LICENSE](./LICENSE).
- **Historique de dépense** : suivi jour par jour (stockage local JSONL,
  rétention 30 j) avec projection de fin de mois, jour pic, moyenne
  journalière et écart réel/estimé. Free = 7 jours visibles, Pro = illimité.
- **Alertes de budget (Pro)** : notifications système quand la dépense
  **réelle** atteint un budget mensuel, journalier ou par provider (une seule
  alerte par période). Réglages › Alertes, avec bouton de test.
- **Assistant de premier lancement** : connecte un provider et teste la clé
  en direct en moins d'une minute (Gemini renvoyé vers la config avancée).
- **Internationalisation** : interface en **français, anglais, allemand et
  espagnol** (100 clés chacune), avec détection automatique de la langue du
  système (repli français) et sélecteur de langue dans Réglages › Préférences.
  Framework `src/i18n/` (dictionnaires JSON + `t()`), parité de clés et de
  variables vérifiée par tests.
- **Licence Pro freemium** via Lemon Squeezy : Free = 1 provider, Pro =
  providers illimités. Activation/validation/désactivation en ligne (1 poste
  par clé, grâce hors-ligne 7 j, revalidation quotidienne, révocation auto au
  remboursement). Bandeau Free/Pro, onglet « Licence & compte », cartes
  provider verrouillées + sélecteur de provider actif en Free.
- **Variante de build « perso »** (`build:perso` / `build:all`) incluant le
  suivi des abonnements Max claude.ai (flag `aiumPersonal` baké) ; le build
  vendu reste sans `src/personal`.

### Changed

- Licence du dépôt : **propriétaire** (non open source) au lieu de MIT.

### Removed

- Essai 15 jours signé Ed25519 (remplacé par le freemium ; les modules de
  vérification Ed25519 sont conservés, inutilisés).

### Changed

- Bandeau licence/essai redessiné (`.licbar`) : icône, jauge de jours
  restants et bouton « Activer » à la place du message texte nu.
- Période d'essai portée à 15 jours.

### Fixed

- Message clair quand la session claude.ai a expiré (challenge Cloudflare) :
  « Session claude.ai expirée — reconnecte-toi » au lieu de `invalid_key`.
- Échappement du libellé dans les info-bulles du donut (cohérence XSS).
- **Jauges de limites Max** ne s'affichaient pas : les endpoints claude.ai
  `/usage` et `/rate_limits` sont derrière un challenge Cloudflare qui
  renvoyait un 403 (« Just a moment… ») à `fetch` Node (vu comme
  `invalid_key`). Les appels claude.ai passent désormais par `net.fetch` sur
  la session `persist:claudeai` (cookies `cf_clearance` + `sessionKey` du
  navigateur de login), ce qui franchit Cloudflare.
- Connexion claude.ai : dump diagnostique (`claude-max-debug.json`) et
  retour d'état enrichi pour identifier pourquoi les jauges de limites Max
  ne s'affichent pas (nombre de jauges lues / erreur / chemin du diag).

### Added

- Widget compact Electron toujours au premier plan, réductible dans le
  tray, résumant l'usage IA en un coup d'œil.
- **Cœur générique multi-provider** (`src/providers/`) : architecture
  `Provider → Source → NormalizedUsage`, contrats validés par
  `validateSource` / `validateNormalizedUsage`, registre de providers
  (`src/providers/registry.js`) pilotant le widget et le tableau de bord.
- Provider **Anthropic** avec deux sources : API Console (Admin API,
  `costBasis: "actual"`) et Claude Code local (`costBasis: "estimated"`).
- Provider **OpenAI** : source API org costs/usage (Admin key `sk-admin…`,
  dépense `$` réelle).
- Provider **OpenRouter** : source activity + credits (clé standard,
  dépense `$` réelle, repli sur le total cumulé si l'API n'expose pas le
  détail journalier).
- Provider **Gemini** : source Cloud Monitoring (tokens × tarifs,
  `estimated`) et source BigQuery Billing export (coût réel, `actual`,
  optionnelle).
- **Champs de credentials déclaratifs** (`credentialFields` sur `Source`) :
  saisie multi-champs dans Réglages (JSON compte de service, projet GCP,
  dataset/table BigQuery) ; secrets et config non-secrète stockés
  séparément.
- Chiffrement des secrets adossé à l'OS (Electron `safeStorage` / DPAPI sous
  Windows) : chaque clé / JSON est chiffré avant d'atteindre le disque.
- Migration transparente des secrets existants (v1/v2) vers le chiffrement OS
  au premier lancement.
- Avertissement dans Réglages si le chiffrement OS est indisponible sur la
  machine (secrets stockés en clair, préfixe `plain:`).
- Activation par clé de licence signée (Ed25519, vérifiée hors-ligne) :
  bloc Licence dans Réglages (statut + saisie de clé).
- Période d'essai de 14 jours puis rappel non bloquant (aucune fonction
  désactivée).
- Mises à jour automatiques via `electron-updater` (installeur NSIS, feed
  GitHub).
- Outils dev `scripts/gen-keypair.js` / `scripts/gen-license.js` pour générer
  la paire de clés et signer des licences.
- Onglet **« Vue d'ensemble »** (par défaut) agrégeant tous les providers :
  totaux réel/estimé séparés, dépense par provider, timeline byDay avec
  bascule **$ / tokens** (corrige le mélange d'échelle), table top modèles.
- Onglet **Sessions** : suivi par session Claude Code locale (tokens + coût
  estimé, projet, dernière activité), 20 sessions les plus récentes.
- Widget : barre de déplacement, fenêtre redimensionnable, et affichage des
  **seuls providers actifs** (les autres s'activent depuis Réglages).
- Source **perso (opt-in)** : jauges de limites d'abonnement Max (claude.ai)
  dans le widget — Session 5h, Weekly, Extra Usage… (% + compte à rebours
  reset). Désactivée par défaut (`AIUM_PERSONAL=1` pour l'activer), exclue de
  toute version vendue.
- Suivi de l'API Console via l'Anthropic Admin API (`usage_report` /
  `cost_report`) : dépense `$` réelle, agrégation par jour / modèle /
  workspace, périodes Jour/Semaine/Mois.
- Suivi de l'usage Claude Code local : scan des logs
  `~/.claude/projects/**/*.jsonl`, agrégation par jour / modèle / projet,
  coût estimé via table de prix éditable.
- Stockage chiffré des identifiants (clé par `sourceId`, générique par
  provider) via `electron-store`.
- Réglages : thème clair/sombre, seuils d'alerte, intervalle de
  rafraîchissement, démarrage automatique, table de prix éditable.
- Suite de tests unitaires Vitest (providers/types, providers/registry,
  providers/anthropic/admin-source, providers/anthropic/local-source,
  store, scanner, aggregate, pricing, format, api-console client) — 42
  tests, 10 fichiers.
- Configuration de packaging Windows (electron-builder, cible NSIS),
  packageant désormais `src/providers/**`.
- Documentation : `README.md`, `INSTALL.md`, `QUICKSTART.md`.

### Changed

- Tableau de bord : onglets désormais générés depuis le registre de
  providers plutôt que codés en dur par fournisseur.
- Total agrégé (`grandTotal`) : ne somme plus que les sources
  `costBasis: "actual"` ; les sources `estimated` sont exposées à part
  (`grandTotalEstimated`) pour ne jamais mélanger réel et estimé.

### Security

- La source claude.ai (scraping du panneau d'usage) est **off par défaut** et
  absente du build vendu ; le cookie `sessionKey` est chiffré par l'OS, jamais
  loggé ni exposé au renderer. Endpoint privé non contractuel — usage perso.
- Émission de licences : seule la **clé publique** est committée
  (`src/license/public-key.js`) ; la clé privée d'émission reste dans
  `secrets/` (git-ignoré). Vérification hors-ligne, fail-closed (tout doute →
  non licencié).
- Les secrets ne sont plus stockés en clair : la clé embarquée
  `electron-store` n'est plus la frontière de sécurité (obfuscation de
  fichier au repos uniquement) ; `safeStorage` (clé fournie par l'OS,
  périmètre utilisateur) chiffre désormais les valeurs secrètes. Un jeton
  `enc:v1:` corrompu (ex. fichier copié entre machines) déchiffre en `null`
  → la source repasse « non configurée ».
- JSON du compte de service GCP stocké via le chemin credential (jamais
  loggé ni renvoyé au renderer — seul un booléen `hasValue` transite).
- Rôles GCP au moindre privilège documentés : `monitoring.viewer` pour la
  source Cloud Monitoring, `bigquery.dataViewer` pour la source Billing.

### Removed

- **Suivi de l'abonnement Max (`claude.ai`)** : login via fenêtre
  BrowserWindow cachée, usage de session/hebdomadaire, comptes à rebours
  de reset, onglet Abonnement du tableau de bord, `src/subscription/`.
  Pivot produit : recentrage sur les API officielles documentées
  (Admin API Anthropic aujourd'hui, autres providers officiels à venir)
  plutôt qu'un scraping non contractuel de l'API web `claude.ai`.
