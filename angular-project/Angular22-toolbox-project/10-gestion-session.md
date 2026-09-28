[← 09 · Les appels HTTP](09-appels-http.md) · [Sommaire](Readme.md) · [11 · WebSocket →](11-websocket.md)

# La gestion de session

La session répond à la question « qui est connecté ? ». Elle repose sur trois briques : un **service** qui détient l'utilisateur et son **jeton** (la preuve d'identité remise par le serveur à la connexion), une **garde** qui écarte les visiteurs non connectés des pages privées, et un **intercepteur** qui joint le jeton à chaque appel. Cette fiche va au-delà du cours : le projet Pokedev n'a pas de connexion.

## L'essentiel

### 1. Le service de session

`src/app/core/session/session.ts`

```ts
@Service()
export class Session {
  private readonly http = inject(HttpClient);

  // clé versionnée : changer de format = changer de clé (session.v2)
  // storedToken() : lit la clé dans localStorage, null si absente
  readonly token = signal<string | null>(storedToken('session.v1'));
  // user est vide après un rafraîchissement (à recharger via l'API) : seul le jeton fait foi
  readonly user = signal<User | null>(null);
  readonly isLoggedIn = computed(() => this.token() !== null);

  async login(email: string, password: string): Promise<void> {
    const { token, user } = await firstValueFrom(
      this.http.post<LoginResponse>('/api/login', { email, password }),
    );
    localStorage.setItem('session.v1', JSON.stringify({ token }));
    this.token.set(token);
    this.user.set(user);
  }

  logout(): void {
    localStorage.removeItem('session.v1');
    this.token.set(null);
    this.user.set(null);
  }
}
```

L'état de session tient dans des **signaux** : templates et gardes le lisent, et réagissent quand il change.

### 2. La garde d'authentification

Si l'utilisateur n'est pas connecté, la garde renvoie une `UrlTree` qui le redirige vers la page de connexion :

`src/app/core/session/auth-guard.ts`

```ts
export const authGuard: CanActivateFn = () => {
  const session = inject(Session);
  return session.isLoggedIn() ? true : inject(Router).parseUrl('/login');
};
```

On l'attache à chaque route privée :

```ts
{ path: 'admin', canActivate: [authGuard], loadComponent: () => import('./features/admin/admin-page').then((m) => m.AdminPage) },
```

### 3. L'intercepteur de jeton

Chaque requête sortante reçoit l'en-tête `Authorization` si un jeton existe. Une requête est immuable : on la `clone()` pour la modifier.

`src/app/core/session/token-interceptor.ts`

```ts
export const tokenInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(Session).token();
  return next(token ? req.clone({ setHeaders: { Authorization: `Bearer ${token}` } }) : req);
};
```

### 4. Le circuit complet

```mermaid
flowchart TD
    A[Login] --> B[Jeton stocké]
    B --> C{Garde sur /admin}
    C -->|non connecté| D[Redirection /login]
    C -->|connecté| E["Intercepteur : Authorization Bearer"]
    E --> F[Serveur]
    F -->|401| G[Session expirée — logout]
```

Une garde améliore l'expérience, elle ne **sécurise** rien : c'est le serveur qui vérifie le jeton. Côté client, une réponse `401` (jeton expiré ou invalide) doit déconnecter proprement l'utilisateur.

## Pièges courants

- **Stocker le mot de passe** : jamais, seulement le jeton. Idéalement dans un **cookie httpOnly** (inaccessible au JavaScript) si le serveur le permet : `localStorage` est lisible par n'importe quel script injecté (faille XSS).
- **Oublier une partie de l'état à la déconnexion** : `logout()` efface le jeton stocké, le signal du jeton **et** l'utilisateur.
- **Croire que la garde protège les données** : l'API reste accessible sans passer par l'interface — le serveur doit vérifier le jeton à chaque requête.

## Approfondir

- [Sécurité — angular.dev](https://angular.dev/best-practices/security) (en anglais)
- Cours Angular : chapitres [06 · Services, injection et HTTP](https://github.com/DiginamicAcademy/Angular/blob/main/06-services-http.md) et [09 · État, persistance et architecture](https://github.com/DiginamicAcademy/Angular/blob/main/09-etat-persistance-architecture.md)

---

[← 09 · Les appels HTTP](09-appels-http.md) · [Sommaire](Readme.md) · [11 · WebSocket →](11-websocket.md)
