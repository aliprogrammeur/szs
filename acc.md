# Guide d’intégration de l’accordeon

Ce document explique comment utiliser correctement le composant `ui-accordion` partagé dans un futur feature. Il décrit l’architecture, les entrées / sorties de chaque composant, et montre comment personnaliser en particulier l’en‑tête et le contenu.

## Architecture générale

Le composant est en fait une **double couche de wrapper** autour des composants Angular Material :

```
UiAccordionComponent (wrappe MatAccordion)
    ↓
    gère le mode ('single' ou 'multiple') et ferme automatiquement les autres éléments en mode single
    ↓
UiAccordionItemComponent (wrappe MatExpansionPanel)
    ↓
    MatExpansionPanel (gestion d’animations, accessibilité, état)
```

- `UiAccordionComponent` dispose les items et impose le comportement `single/multiple`.
- `UiAccordionItemComponent` fournit une interface plus simple et des entrées pour la sélection, le template d’en‑tête, etc.

## Entrées de `UiAccordionComponent`

| Propriété | Type | Valeur par défaut | Rôle |
|-----------|------|-------------------|------|
| `mode` | `'single'` ou `'multiple'` | `'multiple'` | Un seul item ouvert ou plusieurs |
| `variant` | `'default'`/`'bordered'`/`'elevated'` | `'default'` | Apparence visuelle |
| `ariaLabel` | string/null | null | Label accessible pour le conteneur |

Exemple :
```html
<ui-accordion [mode]="'single'" [variant]="'bordered'" ariaLabel="Sections">
  <!-- items -->
</ui-accordion>
```

## Entrées et sorties de `UiAccordionItemComponent`

| Input | Type | Usage | Exemple |
|-------|------|-------|---------|
| `headerTemplate` | TemplateRef | template personnalisé | `[headerTemplate]="tpl"` |
| `headerContext` | any | contexte passé au template | `[headerContext]="{name:'foo'}"` |
| `isSelected` | bool | ajoute classe `.ui-accordion-item__header--selected` | `[isSelected]="true"` |
| `disabled` | bool | désactive l’item | `[disabled]="true"` |
| `opened` | bool | item ouvert au départ | `[opened]="true"` |
| `ariaLabel` | string/null | label pour lecteur d’écran | `[ariaLabel]="'Mon panneau'"` |

⚠️ **Output** :
`(openedChange)` émet `'expanded'` ou `'collapsed'` quand l’usager ouvre/ferme.

## Étapes d’intégration

1. **Importer** depuis l’index public, pas depuis les sous-dossiers :
```ts
import { UiAccordionComponent, UiAccordionItemComponent } from 'app/shared/ui/ui-accordion';
```
2. Ajouter ces deux composants à l’array `imports` du composant conteneur.
3. Gérer l’état de sélection **dans le conteneur** (signal, store, route, etc.).
4. Lien HTML de base :
```html
<ui-accordion [mode]="'single'" [variant]="'bordered'">
  <ui-accordion-item [headerTemplate]="basicHeader" [headerContext]="{label:'Titre'}" [isSelected]="selection().section">
    Contenu projeté
  </ui-accordion-item>
</ui-accordion>
```
5. Le `headerTemplate` est obligatoire ; il n’existe plus de propriété `header` simple. Exemple minimal :
```html
<ng-template #basicHeader let-data>
  {{data.label}}
</ng-template>
```

Pour personnaliser davantage :
```html
<ui-accordion-item
  [headerTemplate]="monHeader"
  [headerContext]="{label:'Mes courriers'}">
  …
</ui-accordion-item>

<ng-template #monHeader let-data>
  <span>{{data.label}}</span>
</ng-template>
```
6. Le contenu placé entre les balises `<ui-accordion-item>` est automatiquement projeté (`<ng-content>`).

## Styliser l’en‑tête sélectionné

Quand `isSelected` est vrai, la classe `.ui-accordion-item__header--selected` est ajoutée.
Pour la modifier :
```scss
::ng-deep .ui-accordion .mat-expansion-panel-header
      .ui-accordion-item__header--selected {
  background-color: $color-selection-bg-list !important;
}
```
`::ng-deep` est nécessaire pour traverser l’encapsulation de Material.

## Modes

- `mode="single"` : un seul item ouvert à la fois (ouverture ferme les autres).
- `mode="multiple"` : les items s’ouvrent/ferment indépendamment.

## Erreurs fréquentes

- Importer depuis `.../accordion/accordion.component.ts` au lieu de l’index.
- Tenter de modifier `isSelected` à l’intérieur de l’item.
- Oublier `::ng-deep` pour les styles.
- Tenter de passer `header` (il a été supprimé) ; utilisez `headerTemplate`.
- Ne pas ajouter les composants dans `imports` → « élément inconnu ».

## Exemple concret (mailbox.container.html)

```html
<ng-template #headerTemplate let-folder="$implicit">
  <div class="mailbox-header" [class.selected]="selection().section">
    …
  </div>
</ng-template>

<ui-accordion mode="multiple">
  @for (folder of mailboxes(); track $index) {
    <ui-accordion-item [headerTemplate]="headerTemplate"
                       [headerContext]="folder"
                       [opened]="$index === 0"
                       [isSelected]="$index === 0 && selection().section">
      <app-mailbox-list …></app-mailbox-list>
    </ui-accordion-item>
  }
</ui-accordion>
```

===

Garde ce guide sous la main : il explique l’architecture, les inputs/outputs et la personnalisation de l’en‑tête/du contenu en français comme demandé.
