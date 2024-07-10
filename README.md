# my-package

Ce répo contient une collection de macros utilitaires pour travailler avec Dataform. Les macros sont conçues pour être utilisées dans un projet Dataform et sont importées dans des projets sous forme de packages.


## Installation

Pour utiliser les macros de ce dépôt, vous devez ajouter ce dépôt aux [dépendances](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#dependencies) dans votre fichier `packages.json` :

```json
{
    "name": "my_project",
    "dependencies": {
        "@dataform/core": "^2.0.3",
        "my-package": "https://github.com/quickbi/my-package/archive/main.tar.gz"
    }
}
```
Une fois que vous avez ajouté le package à votre fichier packages.json, vous pouvez installer le package en exécutant la commande suivante :

```bash
dataform install
```

## Macros

Pour utiliser les macros de ce dépôt, vous devez les importer dans votre projet Dataform. Pour importer, ajoutez un fichier includes/my-package.js à votre projet Dataform et importez les macros dans ce fichier. 
Par exemple :

```javascript
// includes/my-package.js
const { deduplicate, generate_surrogate_key } = require("my-package");
module.exports = { deduplicate, generate_surrogate_key };
```

Le nom du fichier inclut sert de namespace pour les macros importées. Autrement dit, si le fichier inclut est nommé `my-package.js`, vous pouvez faire référence aux macros importées en utilisant la syntaxe `${my-package.<macro_name>}`. Par exemple, pour faire référence à la macro deduplicate, utilisez la syntaxe `${my-package.deduplicate(<relation>, <partition_by>, <order_by>)}`.

### deduplicate

Cette macro est utilisée pour dédupliquer les données dans une relation ou une CTE. Elle utilise la fonction fenêtre `row_number()` pour attribuer un numéro de ligne unique à chaque ligne de la table, puis filtre les lignes dont le numéro de ligne est supérieur à 1.


La macro prend un objet avec les propriétés suivantes :

`relation` (string) : Les relations ou CTE à dédupliquer.
`partition_by` (string) : Le champ ou les champs par lesquels partitionner. Plusieurs champs doivent être séparés par des virgules.
`order_by` (string) : Le champ ou les champs par lesquels ordonner. Plusieurs champs doivent être séparés par des virgules.

Usage:

```sql
-- definitions/users.sql

config {
    type: 'table',
    assertions: {
        uniqueKey: ['user_id'],
        nonNull: ['user_id']
    }
}

-- Dédupliquer la table stg_users par user_id, en gardant l'enregistrement le plus récent
${my-package.deduplicate({
    relation: ref('stg_users'),
    partition_by: 'user_id',
    order_by: 'loaded_at desc'
})}
```

```sql
-- definitions/users.sql

config {
    type: 'table',
    assertions: {
        uniqueKey: ['user_id'],
        nonNull: ['user_id']
    }
}

with users as (
  select
    *
  from ${ref('stg_users')}
),

-- Dédupliquer le CTE users par user_id, en gardant l'enregistrement le plus récent
deduplicated as (
  ${my-package.deduplicate({
    relation: 'users',
    partition_by: 'user_id',
    order_by: 'loaded_at desc'
  })}
)

select
  *
from deduplicated
```

### generate_surrogate_key

Cette macro est utilisée pour générer une clé de substitution hachée. Utilisez cette macro pour générer des identifiants uniques pour les tables qui n'ont pas de clé naturelle.

Arguments :

`fields` (array) : Un tableau de champs à hacher.
`default_null_value` (string) : La valeur à utiliser lorsque les champs à hacher sont null. Par défaut, `_my-package_surrogate_key_null`.
Usage :



```sql
-- definitions/stg_exchange_rates.sql

config {
    type: 'table',
    assertions: {
        uniqueKey: ['exchange_rate_id'],
        nonNull: ['exchange_rate_id']
    }
}

select
  ${my-package.generate_surrogate_key(['exchange_rate', 'currency'])} as exchange_rate_id,
  *
from ${ref('raw_exchange_rates')}
```
##