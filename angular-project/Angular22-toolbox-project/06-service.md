[← 05 · computed()](05-computed.md) · [Sommaire](Readme.md) · [07 · Les directives →](07-directive.md)

# Les services

Un service porte la logique et l'état **partagés** : données, appels HTTP, état d'une équipe. L'**injection de dépendances** fournit les instances : quand un composant demande `Team`, Angular lui donne l'instance unique, créée une seule fois et partagée par tous.

## L'essentiel

### 1. @Service() et inject()

`src/app/features/team/team.ts`

```ts
@Service() // ≡ @Injectable({ providedIn: 'root' }) — la forme historique
export class Team {
  private readonly repository = inject(DevRepository); // charge les devs (fiche 09)
  private readonly ids = signal<number[]>([]);

  readonly members = computed(() => this.ids().map((id) => this.repository.byId(id)));
  readonly size = computed(() => this.ids().length);

  toggle(id: number): void {
    this.ids.update((list) =>
      list.includes(id) ? list.filter((i) => i !== id) : [...list, id],
    );
  }
}
```

- `@Service()` (Angular 22) remplace `@Injectable({ providedIn: 'root' })`, que vous croiserez dans du code existant.
- Le service est un **singleton** : une seule instance pour toute l'application, créée à la première demande.
- `inject()` s'appelle à l'initialisation d'un champ : c'est la forme moderne, qui remplace l'injection par le constructeur.

### 2. Demander un service

```ts
export class DexPage {
  private readonly team = inject(Team);     // même instance partout
  protected readonly size = this.team.size; // computed exposé au template
}
```

### 3. Qui fournit quoi

Selon l'endroit où il est fourni, un service est partagé par toute l'application ou propre à un composant :

```mermaid
flowchart TD
    A["app.config.ts — providers de l'application"] --> B[Routeur, HTTP, intercepteurs]
    C["@Service() — providedIn: 'root'"] --> D[Instance unique, à la demande]
    E["Composant — providers du @Component"] --> F[Une instance par composant]
```

### 4. Les valeurs qui ne sont pas des classes

Pour injecter une simple valeur (une URL, un paramètre de configuration), on crée un `InjectionToken` :

```ts
export const API_URL = new InjectionToken<string>('api.url');
// fourniture : { provide: API_URL, useValue: 'https://api.exemple.fr' }
// lecture :     inject(API_URL)
```

## Pièges courants

- **S'étonner de voir `@Injectable`** : c'est la forme historique du même service — les deux cohabitent.
- **Garder dans un composant un état partagé** : dès que deux composants en ont besoin, il passe dans un service.
- **Appeler `inject()` hors contexte d'injection** (dans un callback, une méthode) : Angular lève une erreur. Injectez dans un champ, puis utilisez ce champ.

## Approfondir

- [Injection de dépendances — angular.dev](https://angular.dev/guide/di) (en anglais)
- Cours Angular : chapitre [06 · Services, injection et HTTP](https://github.com/DiginamicAcademy/Angular/blob/main/06-services-http.md)

---

[← 05 · computed()](05-computed.md) · [Sommaire](Readme.md) · [07 · Les directives →](07-directive.md)
