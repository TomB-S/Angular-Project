[← 07 · Les directives](07-directive.md) · [Sommaire](Readme.md) · [09 · Les appels HTTP →](09-appels-http.md)

# Les pipes

Un **pipe** transforme une valeur pour l'affichage : `{{ dev.createdAt | date:'d MMMM y' }}`. C'est le bon endroit pour les formats — dates, nombres, devises — pas pour la logique métier.

## L'essentiel

### 1. Les pipes intégrés

| Pipe | Exemple | Résultat |
|---|---|---|
| `date` | `{{ createdAt \| date:'d MMMM y' }}` | 25 septembre 2026 |
| `number` | `{{ ratio \| number:'1.2-2' }}` | 1,33 |
| `currency` | `{{ price \| currency:'EUR' }}` | 12,00 € |
| `percent` | `{{ score \| percent }}` | 75 % |
| `json` | `{{ dev \| json }}` | le JSON brut (débogage) |

Les formats dépendent de la **locale** (langue et conventions régionales). Pour des dates et des nombres à la française, l'application enregistre la locale `fr` au démarrage (`registerLocaleData` et le token `LOCALE_ID`).

### 2. Un pipe personnalisé

`src/app/shared/dex-number-pipe.ts`

```ts
@Pipe({ name: 'dexNumber' })
export class DexNumberPipe implements PipeTransform {
  transform(value: number): string {
    return dexNumber(value); // délégation à une fonction du domaine
  }
}
```

```html
<span>N° {{ dev.id | dexNumber }}</span>
```

Le pipe délègue à une **fonction du domaine** : la logique reste testable sans Angular, le pipe n'est qu'un adaptateur pour le template. Comme un composant, il s'ajoute aux `imports` de celui qui l'utilise.

### 3. Pur ou impur

Un pipe **pur** (par défaut) ne se recalcule que si son entrée change de **référence**. Un pipe **impur** (`pure: false`) se recalcule à chaque détection de changements : coûteux, il est à réserver aux rares cas qui l'exigent.

## Pièges courants

- **Compter sur un pipe pur pour détecter une mutation interne** : il ne voit que les nouvelles références — comme les signaux.
- **Mettre du métier dans un pipe** : le calcul appartient au domaine, le pipe formate.
- **Oublier la locale** : les dates sortent en anglais si personne n'a enregistré `fr`.

## Approfondir

- [Pipes — angular.dev](https://angular.dev/guide/templates/pipes) (en anglais)
- Cours Angular : chapitre [07 · Directives et pipes](https://github.com/DiginamicAcademy/Angular/blob/main/07-directives-pipes.md)

---

[← 07 · Les directives](07-directive.md) · [Sommaire](Readme.md) · [09 · Les appels HTTP →](09-appels-http.md)
