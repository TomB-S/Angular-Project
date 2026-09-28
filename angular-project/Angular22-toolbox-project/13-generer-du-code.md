[← 12 · Les tests avec Vitest](12-tests-vitest.md) · [Sommaire](Readme.md)

# Générer du code

La CLI d'Angular génère le squelette de chaque brique — composant, directive, pipe, service… — avec son fichier de test, en respectant les conventions de nommage du framework. Cette fiche réunit les commandes, puis un snippet prêt à compléter pour chaque type.

## Les commandes

### 1. ng generate

```bash
ng generate <type> <chemin/nom> [options]
ng g c features/devs/dev-card          # forme courte
```

- `ng g` abrège `ng generate`, et chaque type a son alias : `c` pour `component`, `d` pour `directive`…
- Lancé à la racine du projet, le chemin part de `src/app`.
- Sans CLI installée globalement, préfixez par `npx` : `npx ng g c …`.

### 2. Un type, une commande

| Type | Commande | Crée |
|---|---|---|
| Composant | `ng g c features/devs/dev-card --flat` | `dev-card.ts`, `.html`, `.css` — classe `DevCard` |
| Directive | `ng g d shared/highlight` | `highlight.ts` — classe `Highlight`, sélecteur `[appHighlight]` |
| Pipe | `ng g p shared/initials` | `initials-pipe.ts` — classe `InitialsPipe`, nom `initials` |
| Service | `ng g s features/favorites/favorites` | `favorites.ts` — classe `Favorites`, `@Service()` |
| Garde | `ng g g core/auth` | `auth-guard.ts` — fonction `authGuard` |
| Resolver | `ng g r features/devs/dev` | `dev-resolver.ts` — fonction `devResolver` |
| Intercepteur | `ng g interceptor core/logging` | `logging-interceptor.ts` — fonction `loggingInterceptor` |
| Interface | `ng g i domain/dev` | `dev.ts` — interface `Dev` |

Chaque commande crée aussi un fichier `.spec.ts`, sauf pour l'interface. Composants, directives et services n'ont plus de suffixe (`DevCard`, pas `DevCardComponent`) ; pipes, gardes, resolvers et intercepteurs gardent leur type dans le nom.

Le cours range chaque composant à plat dans son dossier de fonctionnalité, d'où `--flat` : sans cette option, la CLI crée un sous-dossier `dev-card/`.

### 3. Les options utiles

| Option | Effet |
|---|---|
| `--dry-run` (`-d`) | Affiche les fichiers qui seraient créés, sans rien écrire |
| `--flat` | Crée le composant sans sous-dossier (convention du cours) |
| `--skip-tests` | Ne crée pas le fichier `.spec.ts` |
| `--inline-template` (`-t`), `--inline-style` (`-s`) | Template et styles dans le fichier `.ts` |
| `--skip-selector` | Composant sans sélecteur : utile pour une page, jamais écrite comme balise |
| `--implements CanDeactivate` | Choisit le type de garde sans passer par la question interactive |

## Les snippets

Chaque snippet part du fichier généré et y ajoute le minimum utile. Comme dans les autres fiches, les imports TypeScript sont omis.

### 1. Composant

`ng g c features/devs/dev-card --flat`

```ts
@Component({
  imports: [DatePipe], // ce que le template utilise
  selector: 'app-dev-card',
  styleUrl: './dev-card.css',
  templateUrl: './dev-card.html',
})
export class DevCard {
  readonly dev = input.required<Dev>();        // entrée, fournie par le parent
  readonly selected = output<number>();        // sortie, écoutée par le parent
  protected readonly expanded = signal(false); // état local
}
```

```html
<h2>{{ dev().name }}</h2>
<button type="button" (click)="expanded.update((open) => !open)">Détails</button>
@if (expanded()) {
  <p>Membre depuis le {{ dev().createdAt | date }}</p>
}
<button type="button" (click)="selected.emit(dev().id)">Choisir</button>
```

Fiche [02 · Le composant de page](02-composant-page.md).

### 2. Page et route

`ng g c features/devs/dev-page --flat --skip-selector`

```ts
@Component({
  imports: [DevCard],
  styleUrl: './dev-page.css',
  templateUrl: './dev-page.html',
})
export class DevPage {
  private readonly repository = inject(DevRepository);

  readonly id = input.required<number, string>({ transform: numberAttribute }); // paramètre :id
  protected readonly dev = computed(() => this.repository.byId(this.id()));
}
```

Dans `src/app/app.routes.ts` :

```ts
{
  path: 'devs/:id',
  title: 'Fiche dev',
  canActivate: [devIdGuard],
  loadComponent: () => import('./features/devs/dev-page').then((m) => m.DevPage),
},
```

Fiche [03 · Le routeur](03-routeur.md).

### 3. Directive d'attribut

`ng g d shared/highlight`

```ts
@Directive({
  selector: '[appHighlight]',
  host: {
    '[class.highlighted]': 'active()',   // liaison sur l'élément hôte
    '(mouseenter)': 'active.set(true)',  // écoute d'un événement de l'hôte
    '(mouseleave)': 'active.set(false)',
  },
})
export class Highlight {
  protected readonly active = signal(false);
}
```

Utilisation : `<li appHighlight>…</li>`. Fiche [07 · Les directives](07-directive.md).

### 4. Pipe

`ng g p shared/initials`

```ts
@Pipe({ name: 'initials' })
export class InitialsPipe implements PipeTransform {
  transform(value: string): string {
    return initials(value); // fonction du domaine, testable sans Angular
  }
}
```

Utilisation : `{{ dev().name | initials }}`. Fiche [08 · Les pipes](08-pipe.md).

### 5. Service

`ng g s features/favorites/favorites`

```ts
@Service()
export class Favorites {
  private readonly ids = signal<number[]>([]);        // état modifiable, privé
  readonly all = this.ids.asReadonly();               // exposé en lecture seule
  readonly count = computed(() => this.ids().length); // état dérivé

  add(id: number): void {
    this.ids.update((list) => [...list, id]);         // nouvelle référence
  }
}
```

Utilisation : `private readonly favorites = inject(Favorites);`. Fiche [06 · Les services](06-service.md).

### 6. Gardes

`ng g g core/auth`

```ts
export const authGuard: CanActivateFn = () =>
  inject(Session).isLoggedIn() ? true : inject(Router).parseUrl('/login');
```

`ng g g features/devs/unsaved --implements CanDeactivate`

```ts
export const unsavedGuard: CanDeactivateFn<DevEditPage> = (page) =>
  !page.dirty() || confirm('Quitter sans enregistrer ?'); // la page expose un signal dirty
```

Sur la route : `canActivate: [authGuard]`, `canDeactivate: [unsavedGuard]`. Fiches [03 · Le routeur](03-routeur.md) et [10 · La gestion de session](10-gestion-session.md).

### 7. Resolver

`ng g r features/devs/dev`

```ts
export const devResolver: ResolveFn<Dev | undefined> = (route) =>
  inject(DevRepository).byId(Number(route.paramMap.get('id')));
```

Sur la route : `resolve: { dev: devResolver }`. Avec `withComponentInputBinding()`, la donnée arrive dans l'entrée `dev` de la page.

### 8. Intercepteur

`ng g interceptor core/logging`

```ts
export const loggingInterceptor: HttpInterceptorFn = (req, next) => {
  const start = Date.now(); // avant la requête
  return next(req).pipe(
    finalize(() => console.log(`${req.url} : ${Date.now() - start} ms`)), // après, succès ou erreur
  );
};
```

Enregistrement dans `app.config.ts` : `provideHttpClient(withInterceptors([loggingInterceptor]))`. Fiche [09 · Les appels HTTP](09-appels-http.md).

## Pièges courants

- **Répéter le type dans le nom** : `ng g g auth-guard` produit `auth-guard-guard.ts` et `authGuardGuard`. Donnez le nom seul : `ng g g auth`.
- **Croire que la CLI branche ce qu'elle crée** : elle crée les fichiers, rien de plus. Un composant, une directive ou un pipe s'ajoute aux `imports` de celui qui l'utilise ; une garde ou un resolver s'attache à une route ; un intercepteur s'enregistre dans `withInterceptors`.
- **Lancer la commande depuis un sous-dossier** : le chemin part alors de ce dossier. En cas de doute, `--dry-run` montre où les fichiers seront créés.

## Approfondir

- [ng generate — angular.dev](https://angular.dev/cli/generate) (en anglais)
- Cours Angular : chapitre [03 · Les outils et la création du projet](https://github.com/DiginamicAcademy/Angular/blob/main/03-environnement-tooling.md)

---

[← 12 · Les tests avec Vitest](12-tests-vitest.md) · [Sommaire](Readme.md)
