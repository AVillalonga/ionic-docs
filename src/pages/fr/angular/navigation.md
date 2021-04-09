---
previousText: 'Cycle de Vie'
previousUrl: '/docs/angular/lifecycle'
nextText: 'Configuration'
nextUrl: '/docs/angular/config'
contributors:
  - mhartington
---

# Navigation Angular

Ce guide couvre la façon dont le cycle de vie de la page fonctionne dans une application construite avec Ionic et Angular.

Le Routage d'Angular est l'une des bibliothèques les plus importantes d'une application Angular. Sans cela, les applications seraient une seule vue/une seule application contextuelle ou ne seraient pas en mesure de maintenir leur état de navigation lors du rafraîchissement du navigateur. Avec Angular Router, nous pouvons créer des applications riches qui sont linkables et ont des animations riches (quand elles sont jumelées avec Ionic bien sûr). Regardons les bases du routage avec Angular et comment nous pouvons le configurer pour les applications Ionic.

## Une simple route

Pour la plupart des applications, il est souvent nécessaire d'avoir une configuration de l'arborescence des routes. La configuration la plus basique ressemble à ceci :

```typescript
<br />import { RouterModule } from '@angular/router';

@NgModule({
  imports: [
  ...
  RouterModule.forRoot([
    { path: '', component: LoginComponent },
    { path: 'detail', component: DetailComponent },
  ])
  ],
})
```

La plus simple répartition de ce que nous avons ici est une recherche de chemin/composant. Lorsque notre application se charge, le routeur démarre les vues en lisant l'URL que l'utilisateur essaie de charger. Dans notre exemple, notre système de routage recherche `''`, qui est essentiellement notre route index. Pour cela, nous chargeons la vue `LoginComponent`. Juste avant. Cette route correspond à une seule et unique vue pour chaque entrée que nous avons dans la configuration du routeur. Mais que se passe-t-il si nous voulions charger un chemin différent sur notre vue initiale ?

## Gestion des redirections

Pour cela, nous pouvons utiliser les redirections de l'objet routeur. Les redirections fonctionnent de la même manière qu'un objet de route typique, mais ne contiennent que quelques clés différentes.

```typescript
[
  { path: '', redirectTo: 'login', pathMatch: 'full' },
  { path: 'login', component: LoginComponent },
  { path: 'detail', component: DetailComponent }
];
```

Dans notre redirection, nous recherchons le chemin de la page index de notre application. Si nous chargeons cela, nous serons redirigés vers la route `login`. Le paramètre ` pathMatch ` est nécessaire pour dire au système de routage comment il doit faire le routing.

Lorsque nous indiquons `full`, nous disons à l'objet Router que nous devons le comparer avec le chemin complet, même si cela finit par ressembler à `/route1/route2/route3`. C'est à dire que si nous avons :

```typescript
{ path: '/route1/route2/route3', redirectTo: 'login', pathMatch: 'full' },
{ path: 'login', component: LoginComponent },
```

Et nous serons donc redirigés vers `/route1/route2/route3`. Mais si nous chargeons `/route1/route2/route4`, il n'y aura pas de redirection, car les chemins ne sont pas identiques.

Alternativement, si nous utilisons :

```typescript
{ path: '/route1/route2', redirectTo: 'login', pathMatch: 'prefix' },
{ path: 'login', component: LoginComponent },
```

Alors si nous chargeons `/route1/route2/route3` et `/route1/route2/route4`, nous serons bel et bien redirigés vers les deux routes. Ceci est possible parce que `pathMatch : 'prefix'` ne correspondra qu'à une partie du chemin.

## Naviguer vers différentes routes

Parler de routes est bon et tout, mais comment naviguer réellement pour ces routes? Pour cela, nous pouvons utiliser la directive `'routerLink'`. Revenons-en et reprenons notre configuration de l'objet Router utilisé depuis le début :

```ts
RouterModule.forRoot([
  { path: '', component: LoginComponent },
  { path: 'detail', component: DetailComponent }
]);
```

Maintenant depuis le `LoginComponent`, nous pouvons utiliser le code HTML suivant pour naviguer vers la route `detail`.

```html
<ion-header>
  <ion-toolbar>
    <ion-title>Se connecter</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content class="ion-padding">
  <ion-button [routerLink]="['/detail']">Aller au détail</ion-button>
</ion-content>
```

La partie importante ici est `ion-button` et la directive `routerLink`. RouterLink fonctionne sur une idée similaire à celle typique de `href`s, mais au lieu de compiler l'URL en tant que chaîne, elle peut être compilée comme un tableau, ce qui peut fournir des chemins plus compliqués.

Nous pouvons également naviguer par la programmation dans notre application en utilisant L'API du routeur.

```typescript
import { Composant } from '@angular/core';
import { Routeur } from '@angular/router';

@Component({
...
})
export class LoginComponent {

  constructor(private router: Router){}

  navigate(){
    this.router.navigate(['/detail'])
  }
}
```

Les deux options fournissent le même mécanisme de navigation, juste en ajustant différents cas d'utilisation.

> Remarque sur la navigation avec des URL relatives: actuellement, pour prendre en charge plusieurs piles de navigation, les URL relatives ne sont pas prises en charge

## Routes de chargement paresseux

Maintenant, la façon dont nos routes sont configurés fait en sorte qu'ils sont inclus dans le même morceau que le module d'application racine, ce qui n'est pas idéal. Au lieu de cela, le routeur a une configuration qui permet aux composants d'être isolés dans leurs propres morceaux.

```typescript
<br />import { RouterModule } from '@angular/router';

@NgModule({
  imports: [
  ...
  RouterModule.forRoot([
    { path: '', redirectTo: 'login', pathMatch: 'full' },
    { path: 'login', loadChildren: () => import('./login/login.module').then(m => m.LoginModule) },
    { path: 'detail', loadChildren: () => import('./detail/detail.module').then(m => m.DetailModule) }
  ])
  ],
})
```

Bien que similaire, la propriété `loadChildren` est un moyen de référencer un module en utilisant l'importation native au lieu d'un composant directement. Pour ce faire, nous devons cependant créer un module pour chacun des composants.

```typescript
...
import { RouterModule } from '@angular/router';
import { LoginComponent } from './login.component';

@NgModule({
  imports: [
  ...
  RouterModule.forChild([
    { path: '', component: LoginComponent },
  ])
  ],
})
```

> Nous excluons du contenu supplémentaire et n'incluons que les parties nécessaires.

Ici, nous avons une configuration typique de Module Angular, avec une importation de RouterModule, mais nous utilisons maintenant `forChild` et déclarons le composant dans cette configuration. Avec cette configuration, lorsque nous exécuterons notre build, nous produirons des morceaux distincts pour le composant d'application, le composant de connexion et le composant de détail.

## Exemple en direct

Si vous préférez mettre la main sur les concepts et le code décrits ci-dessus, consultez notre [exemple en direct](https://stackblitz.com/edit/ionic-angular-routing?file=src/app/app-routing.module.ts) des sujets ci-dessus sur StackBlitz.

## Travailler avec les onglets

Avec les onglets, le routeur Angular fournit à Ionic le mécanisme permettant de savoir quels composants doivent être chargés, mais le gros du travail est en fait effectué par le composant onglets. Prenons un exemple simple.

```ts
const routes: Routes = [
  {
    path: 'tabs',
    component: TabsPage,
    children: [
      {
        path: 'tab1',
        children: [
          {
            path: '',
            loadChildren: () => import('../tab1/tab1.module').then(m => m.Tab1PageModule)
          }
        ]
      },
      {
        path: '',
        redirectTo: '/tabs/tab1',
        pathMatch: 'full'
      }
    ]
  },
  {
    path: '',
    redirectTo: '/tabs/tab1',
    pathMatch: 'full'
  }
];
```

Ici, nous avons un chemin "tabs" que nous chargeons. Dans cet exemple, nous appelons le chemin "tabs", mais le nom des chemins peut être modifié. Ils peuvent être appelés de la manière qui convient à votre application. Dans cet objet route, nous pouvons également définir une route enfant. Dans cet exemple, la route enfant de premier niveau "tab1" fait office de "sortie" et peut charger des routes enfant supplémentaires. Pour cet exemple, nous avons une seule route de sous-filière, qui charge simplement un nouveau composant. Le balisage de l'onglet est le suivant :

```html
<br /><ion-tabs>

  <ion-tab-bar slot="bottom">

    <ion-tab-button tab="tab1">
      <ion-icon name="flash"></ion-icon>
      <ion-label>Tab One</ion-label>
    </ion-tab-button>

  </ion-tab-bar>

</ion-tabs>
```

Si vous avez déjà construit des applications avec Ionic, cela devrait vous sembler familier. Nous créons un composant `ion-tabs`, et fournissons une `ion-tab-bar`. Le `ion-tab-bar` fournit un `ion-tab-button` avec une propriété `tab` qui est associée à l'onglet "outlet" dans la configuration du routeur. Notez que la dernière version de `@ionic/angular` ne nécessite plus `<ion-tab>`, mais permet plutôt aux développeurs de personnaliser entièrement la barre d'onglets, et la source unique de vérité vit dans la configuration du routeur.