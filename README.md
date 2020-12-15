# Custom Blog - création d’un extension WordPress

## WordPress plugin

Nous allons utiliser @wordpress/create-block un package qui génère un simple plugin WordPress qui ajoute un simple block à l’éditeur. Cette approche, inspirée par `create-react-app`, est recommandée par WordPress.

La commande `npx @wordpress/create-block` génère du code PHP, JS, CSS indispensable pour démarrer le projet et enregistrer un plugin WordPress. Il prend aussi en charge toute la configuration nécessaire.

👉 Dans votre instance de WordPress en local, déplacez-vous dans le répertoire des extenstions `wp-content/plugins` et installez `@wordpress/create-block` :

```bash
cd wp-content/plugins
npx @wordpress/create-block our-sponsors
cd our-sponsors
```

```bash
├── block.json
├── build
│   ├── index.asset.php
│   ├── index.css
│   ├── index.js
│   └── style-index.css
├── node_modules
├── our-sponsors.php
├── package-lock.json
├── package.json
├── readme.txt
└── src
    ├── edit.js
    ├── editor.scss
    ├── index.js
    ├── save.js
    └── style.scss
```

Extensions (plugins) WordPress ont pour but d'ajouter des fonctionnalités. Dans notre cas, le plugin `our-sponsors` ajoute un nouvel block au sein de l’éditeur.

Techniquement, un plugin est un répertoire (ou un fichier simple) qui :

- est placé dans `wp-content/plugins`
- contient un fichier `.php` qui commence par une structure de commentaires , qui permet au WordPress de reconnaître le plugin.

👉 Nous allons modifier la partie entête de fichier `our-sponsors.php`

```php
<?php
/**
 * Plugin Name:     Our Sponsors
 * Description:     Simple block that adds some information about the article sponsors.
 * Version:         0.1.0
 * Author:          Paulina Hetman
 * License:         GPL-2.0-or-later
 * License URI:     https://www.gnu.org/licenses/gpl-2.0.html
 * Text Domain:     our-sponsors
 *
 * @package         create-block
 */
```

Nous retrouvons notre plugin dans le dashboard de WordPress (Extensions installées)

![](https://paper-attachments.dropbox.com/s_F45F85F9387024D6F24B7C73EA6CDAAB2433290EEB9CB765965C08123927E256_1608015303016_All-in-One+WP+Migration.png)

Nous allons activer _Our Sponsors_. Ensuite nous allons le tester en créant un nouvel article. Effectivement, un nouveau block il y est disponible 🎉.

![](https://paper-attachments.dropbox.com/s_F45F85F9387024D6F24B7C73EA6CDAAB2433290EEB9CB765965C08123927E256_1608015323605_Visibilite.png)

---

## L’anatomie d’un Block

Un block Gutenberg est un objet JavaScript avec un nombre de propriétés et méthodes spécifiques.

```js
// src/index.js
registerBlockType("create-block/our-sponsors", {
  /**
   * Le titre de ce block, affiché dand l'éditeur, il peut être traduit
   */
  title: __("Our Sponsors", "our-sponsors"),
  /**
   * La courte description pour ce block, affichée dans le barre latérale. Elle peut être traduite.
   */
  description: __(
    "Simple block that adds some information about the article sponsors.",
    "our-sponsors"
  ),
  /**
   * Les Blocks sont groupés par catégories  (`common`, `embed`, `formatting`, `layout` ou `widgets`).
   */
  category: "widgets",
  /**
   * L'icône affichée pour ce block, https://developer.wordpress.org/resource/dashicons/
   */
  icon: "smiley",
  /**
   * Optional block extended support features.
   */
  supports: {
    // Removes support for an HTML mode.
    html: false,
  },
  /**
   * méthode edit, Edit importée depuis ./edit.js
   */
  edit: Edit,
  /**
   * méthode save, save importée depuis ./save.js
   */
  save,
})
```

👉 Pour commencer nous allons modifier le block de base afin qu’il affiche le paragraphe suivant :

> The publishing of this content was possible thanks to the support of Alyra.

N’oubliez pas de lancer le serveur de développement :

```bash
npm run start
```

### Fichiers `edit.js` & `save.js`

Nous allons modifier ce qui sera affiché dans l'éditeur :

```js
// src/edit.js
export default function Edit({ className }) {
  return (
    <p className={className}>
      {__(
        "The publishing of this content was possible thanks to the support of",
        "our-sponsors"
      )}{" "}
      <a href="https://alyra.fr">Alyra.</a>
    </p>
  )
}
```

et ce qui sera enregistré dans la base de données quand nous sauvegardons l'article :

```js
// src/save.js
export default function save() {
  return (
    <p>
      {__(
        "The publishing of this content was possible thanks to the support of",
        "our-sponsors"
      )}{" "}
      <a className="sponsor" href="https://alyra.fr">
        Alyra.
      </a>
    </p>
  )
}
```

---

### Fichiers .scss

Nous allons modifier les fichier styles :

```scss
// src/style.scss - s'applique au front et back-end
.wp-block-create-block-our-sponsors {
  a {
    color: inherit;
    text-decoration: underline;
  }
}
```

```scss
// src/editor.scss - s'applique au back-end
.wp-block-create-block-our-sponsors {
  border: 1px dotted black;
  padding: 1rem;
}
```

---

## Attributes du block

Les `attributes` servent à enregistrer les données concernant notre block. Les `attributes` informent WordPress comment interpreter le contenu enregistré dans la base de données : quels sont les éléments statiques, et quelles parties sont personnalisable dans l’éditeur.

👉 Nous allons ajouter ceci dans `registerBlockType`

```js
// src/index.js
attributes: {
  sponsorName: {
    type: "string",
    source: "text",
    selector: ".sponsor",
  },
  sponsorUrl: {
    type: "string",
    source: "attribute",
    selector: ".sponsor",
    attribute: "href",
  },
},
```

(Vous trouverez plus de détails sur la configuration des attributs dans la documentation).

Nous passons les `attributes` et la fonction `setAttributes` en tant que props dans la fonction `Edit`

```js
// src/edit.js
import { TextControl } from "@wordpress/components"

export default function Edit({ className, attributes, setAttributes }) {
  return (
    <p className={className}>
      {__(
        "The publishing of this content was possible thanks to the support of",
        "our-sponsors"
      )}{" "}
      <TextControl
        label={__("Sponsor Name", "our-sponsors")}
        value={attributes.sponsorName}
        onChange={(val) => setAttributes({ sponsorName: val })}
      />
      <TextControl label={__("Sponsor Url", "our-sponsors")} value={attributes.sponsorUrl} type="url" onChange={(val) => setAttributes({ sponsorUrl: val })} />
    </p>
  )
}
```

et ensuite nous allons aussi modifier la fonction `save`

```js
export default function save({ attributes }) {
  return (
    <p>
      {__(
        "The publishing of this content was possible thanks to the support of",
        "our-sponsors"
      )}{" "}
      <a className="sponsor" href={attributes.sponsorUrl}>
        {attributes.sponsorName}
      </a>
    </p>
  )
}
```

---

## Propriété `supports`

Nous pouvons ajouter les contrôles des couleurs :

    supports: {
      html: false,
      colors: true
    }

Vous trouverez plus d’options dans la documentation.

---

## 🇫🇷Traduction

### Introduction théorique

Dans l’informatique, le système gettext permet de séparer la programmation de la traduction.
Comment gettext fonctionne ?

Au cours de la programmation, toutes les chaînes de carectères qui devraient être traduites sont marqués de la façon spéciale `__("I should be translated", "project-text-domain")`.  
Un site WordPress est composé de plusieurs “projets” (thème et plusieurs extensions), `"project-text-domain"` permet de traiter les textes de chaque thème et extension séparément.
✅Dans notre cas, le text domain est `"our-sponsors"` et nous mettons tous nos texte ainsi : `__("I should be translated", "our-sponsors")`
⬇️
👉Premièrement, un fichier modèle (template, fichier POT) est crée. Ce fichier aura l’extension `.pot` (Portable Object Template). Il comprendra tous les chaines de caractères à traduire, extraites de tous les fichiers au sein d’un projet.
⬇️
👉Le fichier POT sera utilisé pour créer les fichier `.po` (Portable Object) pour chaque langue de traduction (par exemple `fr_FR.po`, `de_DE.po`, etc.)
⬇️
Le fichiers `.po` sont compilés en fichiers binaires `.mo` (Machine Object)
⬇️
Les fichiers `.mo` sont utilisés par WordPress pour assembler le document HTML selon la langue du site.
⬇️
Dans le cas des traductions dans les fichiers .js, WordPress a besoin de convertir le fichier `.po` en format JSON

### Loco Translate

Nous allons déjà pris soin de bien marquer nos textes. Afin de générer le fichier .POT et ensuite les fichiers `.po` et `.mo` pour la langue française, nous allons utiliser une extension WordPress Loco Translate.

![](https://paper-attachments.dropbox.com/s_F45F85F9387024D6F24B7C73EA6CDAAB2433290EEB9CB765965C08123927E256_1608015360101_Loco+Translate.png)

Dans les réglages de loco translate nous allons: “Scanner les fichiers JavaScript avec des extensions : js”

![](https://paper-attachments.dropbox.com/s_F45F85F9387024D6F24B7C73EA6CDAAB2433290EEB9CB765965C08123927E256_1608015376045_Reglages+de+Iextension.png)

Ensuite, dans Extensions, nous allons choisir “Our Sponsors” et clicker “Créer un modèle” - nous allons ainsi générer le fichier POT.

![](https://paper-attachments.dropbox.com/s_F45F85F9387024D6F24B7C73EA6CDAAB2433290EEB9CB765965C08123927E256_1608015406355_Extensions+Our+Sponsors.png)

Une fois fichier POT créé, nous allons ajouter une nouvelle langue (Dans Extensions > Our Sponsors cliquer Nouvelle langue)

![](https://paper-attachments.dropbox.com/s_F45F85F9387024D6F24B7C73EA6CDAAB2433290EEB9CB765965C08123927E256_1608015424521_Initializing+new+translations+in+our-sponsors.png)

![](https://paper-attachments.dropbox.com/s_F45F85F9387024D6F24B7C73EA6CDAAB2433290EEB9CB765965C08123927E256_1608015436100_Francais+Updated+decembre+14+2020+259+-+83+translated+6+strings+1+untranslated.png)

Une fois le bouton “Save” appuyé la traduction des chaines de caractères des fichiers .php est prise en compte.

![](https://paper-attachments.dropbox.com/s_F45F85F9387024D6F24B7C73EA6CDAAB2433290EEB9CB765965C08123927E256_1608015445214_Nos+Sponsors.png)

### Création du fichier JSON

La fonction qui met en place les traductions des strings dans les fichiers `.js` est appelé dans `our-sponsors.php`, c’est la fonction `wp_set_script_translations`
La ligne suivante génère le fichier JSON basé sur notre fichier `.po`. Le nom de ce fichier doit correspondre à `{text-domain}-{locale}-{handles}.json` dans notre cas

- text-domain → our-sponsors
- locale → fr_FR
- et handler est le premier argument de la foncion `wp_set_script_translations` (fichier `our-sponsors.php`) → create-block-our-sponsors-block-editor

  npx po2json our-sponsors-fr_FR.po our-sponsors-fr_FR-create-block-our-sponsors-block-editor.json -f jed1.x

Nous devons ensuite ajouter le répertoire où se trouvera notre fichier traduction en tant que le troisième argument de `wp_set_script_translations`

```php
// our-sponsors.php
wp_set_script_translations( 'create-block-our-sponsors-block-editor', 'our-sponsors', plugin_dir_path( __FILE__ ) );
```

`plugin_dir_path( __FILE__ )` retourne le chemin vers le fichier courant (ex. `/Users/ipehaa/Sites/blogtrotter/wp-content/plugins/our-sponsors/`)
