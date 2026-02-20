# TP2 — XML & JSON

Objectif : Manipuler les formats **XML** et **JSON** ainsi que leurs schémas de validation (**XSD** et **JSON Schema**).

---

## Structure du projet

```
tp2-archi-si/
├── ex1/
│   ├── carnet.xml       ← document XML (carnet d'adresses)
│   └── carnet.xsd       ← schéma XML de validation
├── ex2/
│   ├── bank.xml         ← document XML (banque)
│   └── bank.xsd         ← schéma XML avec héritage
├── ex3/
│   └── etudiants.json   ← document JSON (liste d'étudiants)
└── ex4/
    ├── etudiant.json    ← document JSON (dossier étudiant)
    └── schema.json      ← JSON Schema de validation
```

---

## Exercice 1 — Carnet d'adresses XML + XSD

### Objectif
Créer un fichier XML bien formé représentant un carnet d'adresses, puis écrire le schéma XSD qui le valide.

### Logique de construction du XML

Un carnet contient plusieurs `<contact>`. Chaque contact possède :
- un attribut `type` → `personne`, `entreprise` ou `autre`
- `<nom>` toujours présent
- `<prenom>` **uniquement pour les personnes**
- `<telephone>`
- `<adresse>` → sous-éléments : `<rue>`, `<numero>`, `<ville>`, `<codePostal>`, `<pays>`

```xml
<contact type="personne">
  <nom>Dupont</nom>
  <prenom>Alice</prenom>
  <telephone>06 12 34 56 78</telephone>
  <adresse>
    <rue>Avenue des Fleurs</rue>
    <numero>12</numero>
    <ville>Lyon</ville>
    <codePostal>69001</codePostal>
    <pays>France</pays>
  </adresse>
</contact>
```

### Logique du XSD

| Besoin | Outil XSD |
|---|---|
| `prenom` optionnel | `minOccurs="0"` |
| `type` avec valeurs fixes | `xs:enumeration` |
| `numero` entier | `type="xs:integer"` |
| Réutiliser la structure adresse | `xs:complexType name="AdresseType"` |

### Validation
Ouvrir `carnet.xml` dans **EditiX** (ou tout éditeur XML) → lancer la validation XSD.

---

## Exercice 2 — Schéma XSD pour `bank.xml` (avec héritage)

### Objectif
Écrire un XSD pour un fichier bancaire en utilisant **l'héritage de types**.

### Principe de l'héritage XML Schema

On définit un type de base abstrait `AccountType` que l'on ne peut pas utiliser directement. Les types `SavingsAccountType` et `CheckingAccountType` l'**étendent** via `xs:extension`.

```
AccountType (abstract)
├── balance (> -5000)
└── attribut id (xs:ID)
    ├── SavingsAccountType  → ajoute attribut interest
    └── CheckingAccountType → rien de plus
```

```xml
<!-- Type de base -->
<xs:complexType name="AccountType" abstract="true">
  <xs:sequence>
    <xs:element name="balance"> <!-- contrainte min -5000 --> </xs:element>
  </xs:sequence>
  <xs:attribute name="id" type="xs:ID" use="required"/>
</xs:complexType>

<!-- Héritage -->
<xs:complexType name="SavingsAccountType">
  <xs:complexContent>
    <xs:extension base="AccountType">
      <xs:attribute name="interest" type="xs:decimal" use="required"/>
    </xs:extension>
  </xs:complexContent>
</xs:complexType>
```

### Contraintes importantes

| Règle | Implémentation XSD |
|---|---|
| `id` unique par compte | `type="xs:ID"` |
| `id` unique par client | `type="xs:ID"` |
| `c_id` référence un client | `type="xs:IDREF"` |
| `ac_id` référence un compte | `type="xs:IDREF"` |
| `balance > -5000` | `xs:minExclusive value="-5000"` |

### Validation
Ouvrir `bank.xml` dans EditiX → valider contre `bank.xsd`.

---

## Exercice 3 — Liste d'étudiants JSON

### Objectif
Produire un fichier JSON valide représentant au moins 3 étudiants en respectant des règles de format.

### Structure d'un étudiant

```json
{
  "identifiant": "E001",
  "nom": "Dupont",
  "prenom": "Alice",
  "dateNaissance": "15/03/02",
  "lieuNaissance": "Lyon",
  "adresse": {
    "numRue": 12,
    "nomRue": "Avenue des Fleurs",
    "codePostal": "69001",
    "ville": "Lyon"
  },
  "mail": "alice.dupont@etudiant.fr",
  "coursSuivis": ["Algorithmique", "Base de données"]
}
```

### Règles à respecter

| Propriété | Règle |
|---|---|
| `identifiant`, `nom`, `prenom`, `mail` | **Obligatoires** |
| `dateNaissance` | Format `JJ/MM/AA` |
| `codePostal` | Exactement **5 chiffres** (string) |
| `coursSuivis` | Facultatif, mais si présent → **au moins 1 item** |

### Validation
Coller le contenu sur **http://jsonlint.com/** → doit retourner `Valid JSON`.

---

## Exercice 4 — JSON Schema

### Objectif
Écrire un JSON Schema (`schema.json`) qui valide le document `etudiant.json`.

### Document à valider

```json
{
  "codePerm": "DOEJ01020300",
  "nom": "Doe",
  "prenom": "John",
  "cours": [
    { "sigle": "INF0101", "credits": 3 },
    { "sigle": "INF0102", "credits": 3, "reprise": true }
  ]
}
```

### Structure du schéma

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["codePerm", "nom", "prenom", "cours"],
  "properties": {
    "cours": {
      "type": "array",
      "items": { "$ref": "#/definitions/Cours" }
    }
  },
  "definitions": {
    "Cours": {
      "type": "object",
      "required": ["sigle", "credits"],
      "properties": {
        "sigle":   { "type": "string" },
        "credits": { "type": "integer", "minimum": 1 },
        "reprise": { "type": "boolean" }
      },
      "additionalProperties": false
    }
  }
}
```

### Points clés du schéma

| Règle | Implémentation |
|---|---|
| `sigle` et `credits` toujours présents | dans `required` de `Cours` |
| `reprise` facultative | absente de `required` |
| Pas de propriété inconnue dans un cours | `additionalProperties: false` |
| Réutilisabilité | définition via `$ref` et `definitions` |

### Validation
Coller `etudiant.json` et `schema.json` sur le validateur : **https://json-schema-validator.herokuapp.com/**

---

## Outils recommandés

| Outil | Usage |
|---|---|
| [EditiX](https://www.editix.com/) | Validation XML / XSD |
| [jsonlint.com](http://jsonlint.com/) | Validation JSON |
| [json-schema-validator](https://json-schema-validator.herokuapp.com/) | Validation JSON Schema |
