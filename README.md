# voxfactura-mcp

Branche ton business **VoxFactura** sur ton assistant IA (Claude Desktop, Claude
Code, ou tout client MCP). Tu poses tes questions en langage naturel :

- « Quelles sont mes factures impayées ? »
- « Quelle est ma marge avec le client Martin ? »
- « Combien j'ai dépensé chez Point P ce mois-ci ? »
- « Donne-moi mon récapitulatif TVA du 1er trimestre. »
- « Crée un devis brouillon pour le client 12 : 3 radiateurs à 450. »

L'assistant consulte tes données et peut préparer des **brouillons** ; il ne
**n'envoie jamais** rien à un client (l'envoi reste une validation humaine dans
VoxFactura). Tout passe par une clé API **scopée** que tu contrôles.

## 1. Crée une clé API

Dans VoxFactura : **Réglages → Clés API → Créer une clé**. Coche les permissions
voulues (lecture : factures, dépenses, clients, comptabilité, encaissements ;
écriture : créer des devis, marquer payé, ajouter des dépenses). Copie la clé
`vf_live_…` : elle n'est affichée qu'une seule fois.

- Pour la **marge d'un client** : factures + dépenses.
- La case « Chantiers et regroupements (ancien) » ne sert plus qu'aux outils
  chantiers, gardés pour compatibilité.
- Pour le **récap TVA / FEC** : comptabilité.

## 2. Installe

```bash
pip install voxfactura-mcp
# ou, sans installer : uvx voxfactura-mcp
```

## 3. Configure ton client MCP

### Claude Desktop

`Réglages → Développeur → Modifier la config` :

```json
{
  "mcpServers": {
    "voxfactura": {
      "command": "voxfactura-mcp",
      "env": { "VOXFACTURA_API_KEY": "vf_live_ta_cle_ici" }
    }
  }
}
```

Redémarre Claude Desktop.

### Claude Code

```bash
claude mcp add voxfactura -e VOXFACTURA_API_KEY=vf_live_ta_cle -- voxfactura-mcp
```

## Variables d'environnement

| Variable | Rôle | Défaut |
|---|---|---|
| `VOXFACTURA_API_KEY` | Ta clé API (obligatoire) | aucune |
| `VOXFACTURA_API_BASE_URL` | URL de l'API | `https://api.voxfactura.com` |
| `VOXFACTURA_DOSSIER` | Clé cabinet : numéro du dossier client lu par défaut | aucun |

## Cabinet comptable : une clé pour tous vos dossiers

Un cabinet d'expertise comptable crée une **clé cabinet** (`vf_cab_…`) depuis
son espace cabinet VoxFactura, page **API**. Elle lit, en lecture seule, les
dossiers des clients qui vous ont donné accès, un dossier à la fois :

- l'outil `dossiers` liste vos dossiers (numéro, raison sociale, SIREN) ;
- chaque outil accepte un paramètre `dossier` (numéro du dossier) ; à défaut,
  le serveur prend `VOXFACTURA_DOSSIER` ;
- les outils d'écriture répondent par un refus : une clé cabinet ne modifie rien ;
- chaque lecture figure dans le journal que voit votre client, qui peut retirer
  l'accès à tout moment.

```json
{
  "mcpServers": {
    "voxfactura-cabinet": {
      "command": "voxfactura-mcp",
      "env": { "VOXFACTURA_API_KEY": "vf_cab_votre_cle", "VOXFACTURA_DOSSIER": "1" }
    }
  }
}
```

## Outils

| Outil | Rôle | Permission |
|---|---|---|
| `dossiers` | Dossiers clients d'une clé cabinet | clé cabinet |
| `factures_impayees` | Factures impayées | factures |
| `factures` | Factures (filtres statut / chantier / période) | factures |
| `facture` | Détail d'une facture | factures |
| `pdf_facture` | PDF Factur-X d'une facture émise (base64) | factures |
| `depenses` | Dépenses avec leur client, devis et facture (filtres client / chantier / catégorie / période) | dépenses |
| `justificatifs_depense` | Justificatifs d'une dépense | dépenses |
| `justificatif` | Fichier d'un justificatif (base64) | dépenses |
| `clients` | Liste / recherche clients | clients |
| `marge_client` | Facturé HT, dépenses et marge d'un client, par travail | factures + dépenses |
| `chantiers` | Liste des chantiers (ancien regroupement) | chantiers |
| `chantier` | Détail d'un chantier (ancien regroupement) | chantiers |
| `marge_chantier` | Ancien : CA facturé − dépenses d'un chantier, préférer `marge_client` | factures + dépenses |
| `recap_tva` | TVA collectée / déductible / nette | comptabilité |
| `journal_ventes` | Journal des ventes d'une période | comptabilité |
| `journal_achats` | Journal des achats d'une période | comptabilité |
| `journal_ventes_csv` | Journal des ventes en CSV | comptabilité |
| `journal_achats_csv` | Journal des achats en CSV | comptabilité |
| `export_fec` | Fichier des écritures comptables de l'année | comptabilité |
| `encaissements` | Factures encaissées d'une période (date, montant, moyen) | payments:read |
| `creer_devis_brouillon` | Crée un devis (brouillon) | devis:write |
| `modeles_devis` | Modèles de devis du compte et leurs variables | devis:write |
| `devis_depuis_modele` | Crée un devis (brouillon) depuis un modèle | devis:write |
| `marquer_facture_payee` | Marque une facture payée | payments:write |
| `ajouter_depense` | Ajoute une dépense | expenses:write |

Tous les outils (sauf `dossiers`) acceptent aussi `dossier`, utile seulement
avec une clé cabinet.

## API sous-jacente

Le serveur n'est qu'un habillage de l'API publique VoxFactura (lecture/écriture
scopée). Doc interactive : `https://api.voxfactura.com/api/v1/pub/docs`.
Voir aussi [`llms.txt`](./llms.txt) et [`examples/`](./examples/).

## Éditeur

VoxFactura est édité par ASTERIO 37, SAS immatriculée au RCS de Paris
(SIREN 107 559 619), 231 rue Saint-Honoré, 75001 Paris. Données hébergées
dans l'Union européenne. Site : https://www.voxfactura.com. Contact :
contact@voxfactura.com. Mentions légales :
https://www.voxfactura.com/legal/mentions-legales.

La facture électronique passe par une plateforme agréée par
l'administration fiscale : VoxFactura n'est pas elle-même plateforme agréée.

## Licence

MIT.
