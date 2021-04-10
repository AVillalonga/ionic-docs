---
---

# Configuration

## Fichiers

Les valeurs de configuration sont stockées dans des fichiers JSON. Le CLI Ionic se base sur un fichier de configuration global, généralement situé ici `~/.ionic/config. son`, et des fichiers de configuration du projet, généralement situé à la racine du projet dans `ionic.config.json`.

Le CLI fournit des commandes pour définir et afficher les valeurs de configuration à partir des fichiers de configuration du projet et du fichier de configuration global CLI. Voir `ionic config --help` ou voir la documentation pour l'utilisation des commandes [`configuration ionic get`](/docs/cli/commands/config-get) et [`configuration ionic set`](/docs/cli/commands/config-set).

### Fichier de configuration du projet

Chaque projet Ionic a un fichier de configuration de projet, généralement situé à la racine. Voir ci-après le fichier `ionic.config.json` commenté.

```json
{
  // Nom de l'application.
  "name": "My App",

  // Le type de projet de l'application. Le CLI utilise cette valeur pour déterminer les commandes
  // et les options disponibles, quoi afficher pour la documentation d'aide
  // et ce qu'il faut utiliser pour les compilations de ressources web et le serveur de développement.
  "type": "angular",

  // L'App ID pour Appflow.
  "id": "abc123",

  // Objet de configuration pour des intégrations telles que Cordova et Capacitor.
  "integrations": {
    "cordova": {
      ...
    }
  },

  // Configuration des hooks --voir la section des hooks ci-dessous pour plus de détails.
  "hooks": {
    ...
  }
}
```

## Variables d'environnement

Le CLI cherchera les variables d'environnement suivantes :

* `IONIC_CONFIG_DIRECTORY`: Le répertoire de la configuration globale du CLI. Par défaut `~/.ionic`.
* `IONIC_HTTP_PROXY`: Si vous êtes derrière un proxy, ceci est l'URL par laquelle toutes les requêtes du CLI vont passer. Voir la section [Utiliser un proxy](./using-a-proxy).
* `IONIC_TOKEN` : Authentification automatique avec [Appflow](https://ionic.io/appflow).

## Options

Les options suivantes modifient le comportement d'une commande exécutée par le CLI.

* `--help`: Au lieu d'exécuter la commande, consultez sa page d'aide.
* `--verbose`: Affiche tous les messages de log à des fins de débogage.
* `--quiet`: N'afficher que les messages de log `WARN` et `ERROR`.
* `--no-interactive` : Désactiver les invites interactives et les sorties fantaisistes. Si le CI ou un terminal non TTY est détecté, le CLI est automatiquement non interactif.
* `--confirm` : activer l'auto-confirmation des invites de confirmation. Attention : le CLI vous invite à ne pas faire quelque chose de potentiellement dangereux. L'auto-confirmation peut avoir des résultats inattendus.

## Crochets

Le CLI peut exécuter des scripts lors de certains événements, comme avant et après les constructions. Pour s'accrocher à la CLI, les [scripts npm](https://docs.npmjs.com/misc/scripts) suivants peuvent être utilisés dans `package.json` :

* `ionic:serve:before` : exécuté avant le démarrage du serveur de dev
* `ionic:serve:after` : exécuté après la fin du serveur de dev
* `ionic:build:before` : exécuté avant le début de la construction d'un actif web
* `ionic:build:after` : exécuté après la fin de la construction d'un actif web
* `ionic:capacitor:run:before` : exécuté pendant `ionic capacitor run` avant que l'ouverture de capacitor ne soit exécutée
* `ionic:capacitor:build:before` : exécuté pendant la `ionic capacitor build` avant que L'ouverture de capacitor ne soit exécuté
* `ionic:capacitor:sync:after` : exécuté pendant `ionic capacitor:sync` après une synchro

Lorsque vous utilisez un script shell pour l'un des crochets, le contexte du crochet est défini dans des variables d'environnement préfixées par `IONIC_CLI_HOOK_CTX_`.

L'exemple suivant montre les variables d'environnement qui sont définies pour le crochet `ionic:capacitor:build`.

```shell
IONIC_CLI_HOOK_CTX_NAME=capacitor:build:before
IONIC_CLI_HOOK_CTX_BUILD_CORDOVA_ASSETS=true
IONIC_CLI_HOOK_CTX_BUILD_ENGINE=browser
IONIC_CLI_HOOK_CTX_BUILD_PROJECT=app
IONIC_CLI_HOOK_CTX_BUILD_TYPE=angular
IONIC_CLI_HOOK_CTX_BUILD_VERBOSE=false
IONIC_CLI_HOOK_CTX_CAPACITOR_APP_ID=io.ionic.starter
IONIC_CLI_HOOK_CTX_CAPACITOR_APP_NAME=ionic-starter-app
IONIC_CLI_HOOK_CTX_CAPACITOR_VERBOSE=false
```

Les crochets peuvent également être définis dans `ionic.config.json`. Définissez un objet `crochets` dans le projet, où chaque clé est le nom du crochet (sans le préfixe `ionique :` ), et la valeur est un chemin vers un fichier JavaScript ou un tableau de chemins.

Dans l'exemple suivant, le fichier est importé et exécuté pendant le crochet `ionic:build:before`.

```json
"hooks": {
  "build:before": "./scripts/build-before.js"
},
```

Les fichiers de crochet JavaScript doivent exporter une seule fonction, qui est passée à un seul argument (`ctx`) chaque fois que le crochet s'exécute.

L'argument est le contexte donné au fichier crochet, qui diffère d'un crochet à l'autre et selon les invocations.

`./scripts/build-before.js`:

```javascript
module.exports = function(ctx) {
  console.log(ctx);
};
```

## Projets multi-applications

<small><em>Disponible en CLI 6.2.0+</em></small>

The Ionic CLI supports a multi-app configuration setup, which involves multiple Ionic apps and shared code within a single repository, or [monorepo](/docs/reference/glossary#monorepo).

> These docs give an overview of the multi-app feature of the Ionic CLI, but don't really go into details for each framework.
> 
> If you're using Angular, please see [this article](https://github.com/ionic-team/ionic-cli/wiki/Angular-Monorepo) for examples.

### Setup Steps

1. Create a directory and initialize a monorepo (see [Project Structure](#project-structure) for full details).
2. Initialize the monorepo as an Ionic multi-app project. This will create a multi-app `ionic.config.json` file. See [Config File](#config-file) for full details.

    ```shell
    $ ionic init --multi-app
    ```

1. Use `ionic start` to create Ionic apps or `ionic init` to initialize existing apps (see [Adding an App](#adding-an-app) for full details).

### Project Structure

In a multi-app project, project structure is flexible. The only requirement is a multi-app `ionic.config.json` file at the root of the repository.

Below is an example setup, where apps in the `apps/` directory are separated from the shared code in the `lib/` directory. Notice the root `ionic.config.json` file and the monorepo's `package.json` file.

<file-tree> <file-tree-directory name="apps"> <file-tree-directory name="myApp" collapsed></file-tree-directory> <file-tree-directory name="myOtherApp" collapsed></file-tree-directory> </file-tree-directory> <file-tree-directory name="lib" collapsed></file-tree-directory> <file-tree-file name="ionic.config.json"></file-tree-file> <file-tree-file name="package.json"></file-tree-file> </file-tree>

### Config File

In a multi-app project, apps share a single `ionic.config.json` file at the root of the repository instead of each app having their own. The multi-app config file contains the configuration for each app by nesting configuration objects in a `projects` object. A default app can be specified using `defaultProject`.

Below is an example file, which corresponds to the file structure above.

```json
{
  "defaultProject": "myApp",
  "projects": {
    "myApp": {
      "name": "My App",
      "integrations": {},
      "type": "angular",
      "root": "apps/myApp"
    },
    "myOtherApp": {
      "name": "My Other App",
      "integrations": {},
      "type": "angular",
      "root": "apps/myOtherApp"
    }
  }
}
```

When a multi-app project is detected, the Ionic CLI will operate under the context of an app configured in the root `ionic.config.json`. Project selection criteria is as follows:

1. If the global CLI option `--project` is specified, the project is looked up by key in the `projects` object. For example, `--project=myApp` will select the `myApp` project.
2. If the CLI detects it is being run within a project path, configured with the `root` key, it will select the matched project. For example, using the CLI within the `apps/myOtherApp/src` directory will select the `myOtherApp` project.
3. If a `defaultProject` is specified in `ionic.config.json`, it will select the specified project when the above criteria is not met.

### Adding an App

Apps can be registered in a multi-app project either by using `ionic start` to create new apps or `ionic init` to initialize existing apps.

#### Using `ionic start`

If a multi-app project is detected during `ionic start`, the CLI will add the app configuration to the root `ionic.config.json` file instead of creating a project-specific one.

Dependency installation can be skipped using `--no-deps` if dependencies are hoisted to the root of the monorepo.

```shell
$ cd apps/
$ ionic start "My New App" --no-deps
```

#### Using `ionic init`

If an app was created in a way other than `ionic start`, for example by using a prebuilt template, use `ionic init` to register the existing app with the multi-app project.

> Make sure the app doesn't have an existing `ionic.config.json`.

```shell
$ cd apps/existing-app/
$ ionic init
```

## Advanced Configuration

### Overriding the Build

Normally, the CLI runs a hard-coded set of commands based on the project type. For example, the standard web asset build for Angular projects is `ng run app:build`. The web asset build can be overridden and `ionic build` can continue to be used by utilizing the `ionic:build` [npm script](https://docs.npmjs.com/misc/scripts). Similarly, the dev server can be overridden by using the `ionic:serve` npm script.

Pay close attention to the flags supplied to the script by the Ionic CLI. Irregularities may occur if options are not respected, especially for livereload on devices.

### Command Options

Command options can be expressed with environment variables. They are normally set with `--opt=value` syntax. The naming of these environment variables follows a pattern: start with `IONIC_CMDOPTS_`, add the command name (replacing any spaces with underscores), add the option name (replacing any hyphens with underscores), and then uppercase everything. Boolean flags (command-line options that don't take a value) can be set to `1` or `0`. Strip the `--no-` prefix in boolean flags, if it exists (`--no-open` in ionic serve can be expressed with `IONIC_CMDOPTS_SERVE_OPEN=0`, for example).

For example, the command options in `ionic cordova run ios -lc --livereload-port=1234 --host=0.0.0.0` can also be expressed with this series of environment variables:

```shell
$ export IONIC_CMDOPTS_CORDOVA_RUN_LIVERELOAD=1
$ export IONIC_CMDOPTS_CORDOVA_RUN_CONSOLELOGS=1
$ export IONIC_CMDOPTS_CORDOVA_RUN_LIVERELOAD_PORT=1234
$ export IONIC_CMDOPTS_CORDOVA_RUN_HOST=0.0.0.0
```

If these variables are set in the environment, `ionic cordova build ios` will use new defaults for its options.

### Telemetry

The CLI sends usage data to Ionic to create a better experience. To disable this functionality, run `ionic config set -g telemetry false`.