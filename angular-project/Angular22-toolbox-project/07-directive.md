[← 06 · Les services](06-service.md) · [Sommaire](Readme.md) · [08 · Les pipes →](08-pipe.md)

# Les directives

Une directive modifie l'apparence ou le comportement d'un élément **existant**, sans créer de balise. On écrit surtout des **directives d'attribut** (`<span appTypeColor>`). Les directives structurelles (`*ngIf`, `*ngFor`), qui ajoutaient ou retiraient des éléments, appartiennent au passé : en Angular 22, le **control flow** (`@if`, `@for`, `@defer`) les remplace.

## L'essentiel

### 1. Une directive d'attribut

`src/app/shared/type-color.ts`

```ts
@Directive({
  selector: '[appTypeColor]',
  host: {
    '[style.--type-color]': 'color()',
    '[attr.data-type]': 'appTypeColor()',
  },
})
export class TypeColor {
  readonly appTypeColor = input.required<DevType>();
  protected readonly color = computed(() => TYPE_COLORS[this.appTypeColor()]);
}
```

```html
<span appTypeColor="frontend">Front-end</span>
```

- Le sélecteur entre **crochets** vise un attribut, pas une balise.
- `host` déclare des liaisons sur l'**élément hôte** (celui qui porte l'attribut) : ici une variable CSS et un attribut `data-*`.
- Comme un composant, la directive reçoit des **entrées** : celle qui porte le nom du sélecteur reçoit la valeur de l'attribut (`"frontend"`).

### 2. hostDirectives

Un composant peut appliquer une directive à sa propre balise, sans que l'utilisateur ait à l'écrire :

```ts
@Component({
  hostDirectives: [{ directive: TypeColor, inputs: ['appTypeColor: type'] }],
})
export class DevBadge {}
// <app-dev-badge type="frontend" /> — TypeColor s'applique à l'hôte
```

`inputs` expose l'entrée de la directive sous un autre nom : ici `type`.

### 3. Le control flow remplace les structurelles

`@if`, `@for`, `@switch`, `@defer` forment le **control flow** intégré (fiche [02 · Le composant de page](02-composant-page.md)). Les anciennes directives structurelles `*ngIf` / `*ngFor` relèvent du **code historique** : à savoir lire, à ne plus écrire.

### 4. Le bon choix

| Besoin | Outil |
|---|---|
| Changer l'apparence d'un élément | directive d'attribut |
| Ajouter ou retirer du contenu | `@if` / `@for` (intégrés) |
| Une balise à soi, avec template | composant |

## Pièges courants

- **Oublier les crochets du sélecteur** : `appTypeColor` sans crochets viserait une balise `<appTypeColor>`, pas un attribut.
- **Oublier `host`** : sans liaisons d'hôte, la directive ne touche pas son élément.
- **Écrire encore `*ngIf` / `*ngFor`** : le control flow les remplace, sans import.

## Approfondir

- [Directives — angular.dev](https://angular.dev/guide/directives) (en anglais)
- Cours Angular : chapitre [07 · Directives et pipes](https://github.com/DiginamicAcademy/Angular/blob/main/07-directives-pipes.md)

---

[← 06 · Les services](06-service.md) · [Sommaire](Readme.md) · [08 · Les pipes →](08-pipe.md)
