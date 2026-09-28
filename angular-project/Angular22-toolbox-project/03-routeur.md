[← 02 · Le composant de page](02-composant-page.md) · [Sommaire](Readme.md) · [04 · signal() →](04-signal.md)

# Le routeur

Le routeur associe chaque URL à une page : l'adresse affiche un composant dans `<router-outlet />`, sans recharger la page. L'URL devient un état de l'application : on peut la partager, la mettre en favori, et elle survit à un rafraîchissement.

## L'essentiel

### 1. La table des routes

`src/app/app.routes.ts`

```ts
export const routes: Routes = [
  { path: '', component: DexPage },
  {
    path: 'devs/:id',
    loadComponent: () => import('./features/devs/dev-page').then((m) => m.DevPage),
  },
  { path: '**', component: NotFoundPage },
];
```

- `:id` déclare un **paramètre** : `/devs/7` affiche `DevPage` avec l'identifiant `'7'`.
- `loadComponent` charge la page **à la demande** (*chargement différé*) : elle est exclue du bundle initial et téléchargée seulement quand l'utilisateur la visite.
- `''` est la route racine ; `'**'` attrape toutes les autres URL : placez-la toujours en **dernier**.

### 2. La navigation

```mermaid
flowchart LR
    A[Clic sur un RouterLink] --> B[Gardes]
    B -->|refus| C[Redirection UrlTree]
    B -->|autorisé| D[Chargement différé du composant]
    D --> E[Affichage dans router-outlet]
```

Le routeur est activé dans `src/app/app.config.ts` :

```ts
provideRouter(routes, withComponentInputBinding()),
```

Dans un template, `routerLink` remplace `href` : la navigation se fait sans recharger la page. `routerLinkActive` ajoute une classe au lien de la page courante.

```html
<a routerLink="/devs" routerLinkActive="active">Pokédex</a>
<a [routerLink]="['/devs', dev().id]">Fiche</a>
```

### 3. Les paramètres comme entrées

Avec `withComponentInputBinding()`, les paramètres d'URL alimentent directement les entrées du composant :

`src/app/features/devs/dev-page.ts`

```ts
export class DevPage {
  readonly id = input.required<number, string>({ transform: numberAttribute });
}
```

Un paramètre d'URL est toujours une chaîne : `numberAttribute` le convertit en nombre. D'où les deux types de `input.required<number, string>` : le type exposé, puis le type reçu.

### 4. Les gardes

| Fonction | Question posée | Réponse |
|---|---|---|
| `CanActivateFn` | Peut-on entrer sur cette route ? | `true`, `false` ou une redirection (`UrlTree`) |
| `CanDeactivateFn` | Peut-on quitter cette page ? | `true` ou `false` |
| `ResolveFn` | Quelle donnée charger avant l'affichage ? | une valeur |

Les gardes sont de simples **fonctions**, qui peuvent appeler `inject()`. Celle-ci redirige vers la page introuvable si l'identifiant n'est pas valide :

`src/app/core/dev-id-guard.ts`

```ts
export const devIdGuard: CanActivateFn = (route) => {
  const id = Number(route.paramMap.get('id'));
  return Number.isInteger(id) && id > 0 ? true : inject(Router).parseUrl('/introuvable');
};
```

On l'attache à la route avec `canActivate: [devIdGuard]`.

## Pièges courants

- **Oublier `provideRouter`** : le routeur ne connaît aucune route, et la navigation échoue avec une erreur dans la console.
- **Placer `'**'` avant les autres routes** : elle les avale toutes.
- **Traiter un paramètre comme un nombre** : c'est une chaîne — convertissez-le (`numberAttribute` en entrée, ou `Number()`).
- **Compter sur une garde pour protéger des données** : une garde améliore l'expérience, elle ne **sécurise** rien — la vraie protection est côté serveur.

## Approfondir

- [Routing — angular.dev](https://angular.dev/guide/routing) (en anglais)
- Cours Angular : chapitre [08 · Routing](https://github.com/DiginamicAcademy/Angular/blob/main/08-routing.md)

---

[← 02 · Le composant de page](02-composant-page.md) · [Sommaire](Readme.md) · [04 · signal() →](04-signal.md)
