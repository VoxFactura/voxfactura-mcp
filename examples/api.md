# Exemples d'appels à l'API publique VoxFactura

Remplace `vf_live_…` par ta clé (Réglages → Clés API).

```bash
BASE="https://www.voxfactura.com/api/v1/pub"
KEY="vf_live_ta_cle"
AUTH="Authorization: Bearer $KEY"
```

## Lecture

```bash
# Factures impayées
curl -s -H "$AUTH" "$BASE/invoices?unpaid_only=true"

# Détail d'une facture (avec lignes)
curl -s -H "$AUTH" "$BASE/invoices/1"

# Marge du client 3 : facturé HT, dépenses, marge, une ligne par travail
curl -s -H "$AUTH" "$BASE/clients/3/margin"

# Dépenses rattachées au client 3 (chacune dit son client, devis et facture)
curl -s -H "$AUTH" "$BASE/expenses?client_id=3"

# Ancien regroupement, gardé pour compatibilité : chantiers en cours
curl -s -H "$AUTH" "$BASE/chantiers?statut=en_cours"

# Récapitulatif TVA du 1er trimestre 2026
curl -s -H "$AUTH" "$BASE/accounting/vat-summary?period_start=2026-01-01&period_end=2026-03-31"

# PDF Factur-X d'une facture émise
curl -s -H "$AUTH" "$BASE/invoices/4/pdf" -o facture-4.pdf

# Factures émises en septembre 2026
curl -s -H "$AUTH" "$BASE/invoices?from=2026-09-01&to=2026-09-30"

# Justificatifs d'une dépense, puis le fichier
curl -s -H "$AUTH" "$BASE/expenses/12/justificatifs"
curl -s -H "$AUTH" "$BASE/expenses/12/justificatifs/<id>" -o ticket.jpg

# Journal des ventes de septembre en CSV
curl -s -H "$AUTH" "$BASE/accounting/sales-journal.csv?period_start=2026-09-01&period_end=2026-09-30" -o ventes.csv

# Encaissements de septembre (permission payments:read)
curl -s -H "$AUTH" "$BASE/payments?from=2026-09-01&to=2026-09-30"

# Export FEC 2026 (téléchargement)
curl -s -H "$AUTH" "$BASE/accounting/fec?year=2026" -o FEC_2026.txt
```

## Écriture (permissions dédiées ; jamais d'envoi client)

```bash
# Créer un devis brouillon
curl -s -H "$AUTH" -H "Content-Type: application/json" \
  -d '{"client_id":3,"objet":"Toiture","lignes":[
        {"designation":"Radiateur","quantite":3,"prix_unitaire_ht":450,"taux_tva":10}]}' \
  "$BASE/devis"

# Modèles de devis, puis un devis brouillon depuis l'un d'eux
curl -s -H "$AUTH" "$BASE/devis/templates"
curl -s -H "$AUTH" -H "Content-Type: application/json" \
  -d '{"modele":"Entretien chaudière","variables":{"date":"2026-11-03"},"client_id":3}' \
  "$BASE/devis/from-template"

# Marquer une facture payée (partiel possible via montant)
curl -s -H "$AUTH" -H "Content-Type: application/json" \
  -d '{}' "$BASE/invoices/1/mark-paid"

# Ajouter une dépense rattachée à un chantier
curl -s -H "$AUTH" -H "Content-Type: application/json" \
  -d '{"designation":"Ciment","montant_ttc":"240.00","categorie":"materiel","chantier_id":42}' \
  "$BASE/expenses"

# Joindre la facture PDF à la dépense n°12 (ligne du relevé) : lue, HT et TVA
# repris si la ligne n'en avait pas. Renvoyer le même fichier ne double rien.
curl -s -H "$AUTH" -F "fichier=@facture.pdf" "$BASE/expenses/12/justificatifs"
```

## Clé cabinet (cabinet comptable, lecture seule)

```bash
KEY_CAB="vf_cab_votre_cle"
AUTH_CAB="Authorization: Bearer $KEY_CAB"

# Vos dossiers clients (numéro, raison sociale, SIREN)
curl -s -H "$AUTH_CAB" "$BASE/cabinet/dossiers"

# Factures émises en septembre 2026 du dossier 3
curl -s -H "$AUTH_CAB" "$BASE/invoices?dossier=3&from=2026-09-01&to=2026-09-30"

# Même chose avec l'en-tête plutôt que le paramètre
curl -s -H "$AUTH_CAB" -H "X-VoxFactura-Dossier: 3" "$BASE/accounting/vat-summary?period_start=2026-07-01&period_end=2026-09-30"
```

## Python (client fourni)

```python
from voxfactura_mcp.client import VoxFacturaClient

c = VoxFacturaClient(api_key="vf_live_ta_cle")
print(c.list_invoices(unpaid_only=True))
print(c.client_margin(3))

# Clé cabinet : un dossier à la fois
cab = VoxFacturaClient(api_key="vf_cab_votre_cle")
for d in cab.list_dossiers():
    print(d["num"], d["raison_sociale"], cab.pour_dossier(d["num"]).list_payments())
```
