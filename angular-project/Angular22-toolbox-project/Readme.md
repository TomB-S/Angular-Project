# Angular 22 — Toolbox

12 fiches notions Angular 22 et un mémo de génération de code : la référence rapide du cours [Angular](https://github.com/DiginamicAcademy/Angular) (CDA).

Une fiche = une notion, résumée à l'essentiel. Pour apprendre Angular en construisant, suivez le cours : le projet Pokedev d'abord, puis Time To en autonomie.

## Sommaire

| # | Fiche | Notion |
|---|---|---|
| 01 | [L'application](01-app.md) | Bootstrap, composant racine, providers, zoneless |
| 02 | [Le composant de page](02-composant-page.md) | `@Component`, template, OnPush, entrées / sorties |
| 03 | [Le routeur](03-routeur.md) | Routes, chargement différé, gardes, paramètres |
| 04 | [`signal()`](04-signal.md) | État réactif : créer, lire, modifier |
| 05 | [`computed()`](05-computed.md) | Valeurs dérivées, mémoïsation |
| 06 | [Les services](06-service.md) | `@Service()`, `inject()`, injection de dépendances |
| 07 | [Les directives](07-directive.md) | Attribut, control flow, personnalisée |
| 08 | [Les pipes](08-pipe.md) | Intégrés, personnalisés, pur / impur |
| 09 | [Les appels HTTP](09-appels-http.md) | `HttpClient`, `httpResource`, intercepteurs |
| 10 | [La gestion de session](10-gestion-session.md) | Authentification, gardes, jeton, intercepteur |
| 11 | [WebSocket](11-websocket.md) | Temps réel, reconnexion, signaux |
| 12 | [Les tests avec Vitest](12-tests-vitest.md) | `ng test`, spec, `TestBed`, couverture |
| 13 | [Générer du code](13-generer-du-code.md) | `ng generate`, options, snippets par type |

Les fiches suivent une progression — chacune s'appuie sur les précédentes — mais chacune se lit seule. La fiche 13 est un mémo, à garder sous la main dès la fiche 02.

## Versions

Angular 22, TypeScript 6, Vitest 4, RxJS 7.8, Node.js 22 ou 24 LTS.

Angular publie une version majeure environ tous les six mois ; `npm view @angular/cli version` affiche la dernière publiée.

## Comment lire une fiche

Chaque fiche suit le même plan :

- **Introduction** — à quoi sert la notion, et quand l'utiliser.
- **L'essentiel** — le code minimal, qui couvre la plupart des usages.
- **Pièges courants** — les erreurs classiques de débutant.
- **Approfondir** — la documentation officielle, et le chapitre du cours qui traite la notion en détail.

Certaines fiches dépassent le cours : les fiches 10 (session) et 11 (WebSocket) entièrement, la fiche 09 en partie (mutations `POST`, transport Fetch). Elles préparent aux applications réelles, en s'appuyant sur les chapitres 06 (services, HTTP, intercepteur) et 09 (état, persistance) du cours.
