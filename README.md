# SkeletonBundle

Comment créer un bundle Symfony >=6.1

## Information

NORMALEMENT, si vous téléchargez ce dépôt, vous n'avez pas besoin de faire les étapes ci-dessous. Vous pouvez
remplacer `Skeleton` par le nom de votre bundle de partout (namespace, routes/config, nom de fichier...), ça devrait
fonctionner tel quel. Have fun!

Si vous pouvez me faire des retours, c'est cool.

## Architecture d'un bundle

https://symfony.com/doc/current/bundles.html#bundle-directory-structure

```
    amu-skeleton-bundle
    ├── config/
        ├── routes/amu_skeleton.yaml <<<= Modèle pour la documentation (n'est jamais chargé automatiquement)
        ├── packages/amu_skeleton.yaml <<<= Modèle pour la configuration (n'est jamais chargé automatiquement)
        ├── services.yaml <<<= Définir ses services (+ alias, ex: '@skeleton_service')
    ├── src <<<= Que du PHP ici
        ├── Controller/
        ├── Service/
        ├── EventSubscriber/
        ├── Entity/
        ├── ...
        ├── AmuSkeletonBundle.php <<<= Charge la configuration et inject les services
    ├── templates/ <<<= html.twig
    ├── composer.json (name: "amu/skeleton-bundle")
```

## Pendant le développement du bundle

Dans votre projet ou dans un nouveau projet Symfony (ici on va utiliser un nouveau projet Symfony)

```sh
symfony new projettestbundle --version="lts" --webapp
cd projettestbundle
mkdir -p bundle/Entreprise/
cd bundle/Entreprise
git clone # + adresse du dépôt du bundle
mv skeletonbundle SkeletonBundle # On renomme le dossier
```

Puis dans `composer.json` du projet Symfony (**Pas celui du bundle**), ajoutez :

```json
    "minimum-stability": "dev",
    "repositories": [
        {
            "type": "path",
            "url": "./bundle/Entreprise/SkeletonBundle"
        }
    ],
```

Puis

```sh
composer require Entreprise/skeleton-bundle
```

Si tout se passe bien, vous pouvez maintenant utiliser votre bundle dans votre application Symfony, faire votre développement, sans avoir à versionner puis `composer update` dans votre application Symfony.

## Tips

- Dans composer.json, je cite la doc symfony :
  `It's recommended to use the PSR-4 autoload standard: use the namespace as key, and the location of the bundle's main
class (relative to composer.json) as value. As the main class is located in the src/ directory of the bundle:`

```json
{
  "autoload": {
    "psr-4": {
      "Entreprise\\SkeletonBundle\\": "src/"
    }
  }
}
```

- Pour les fichiers yaml de services/configuration, je cite la doc
  symfony : `The root key of your bundle configuration (acme_social in the previous example) is automatically determined from your bundle name (it's the snake case of the bundle name without the Bundle suffix).`
  Pour notre exemple, il faut donc utiliser `Entreprise_skeleton` comme clé (important pour le fichier de configuration
  notamment).

- Pour vérifier le format de la configuration (Celle défini dans EntrepriseSkeletonBundle.php)

```sh
symfony console config:dump-reference
```

- Pour vérifier que la configuration du bundle est bien chargé par votre application Symfony

```sh
symfony console debug:config
```

## EntrepriseSkeletonBundle.php

```php
<?php

namespace Entreprise\SkeletonBundle;

use Symfony\Component\HttpKernel\Bundle\AbstractBundle;

class EntrepriseSkeletonBundle extends AbstractBundle {}
```

## Ajouter une configuration

- On définit notre arbre de configuration dans `EntrepriseSkeletonBundle.php`

```php
class EntrepriseSkeletonBundle extends AbstractBundle
{
    public function configure(DefinitionConfigurator $definition): void
    {
        $definition->rootNode()
            ->children()
            ->arrayNode('rules')
            ->children()
            ->integerNode('rule1')->end()
            ->scalarNode('rule2')->end()
            ->end()
            ->end()
            ->end();
    }
```

- On demande à charger la configuration dans `EntrepriseSkeletonBundle.php`

```php
class EntrepriseSkeletonBundle extends AbstractBundle
{
    public function loadExtension(array $config, ContainerConfigurator $containerConfigurator, ContainerBuilder $containerBuilder): void
    {
        // On ajoute notre tableau de configuration à nos parameters
        $containerConfigurator->parameters()
            ->set('entreprise_skeleton_config', $config['rules']);
    }
```

- Créer le fichier `config/packages/entreprise_skeleton.yaml` qui correspond à votre arbre de configuration.

```yaml
entreprise_skeleton:
  rules:
    rule1: 1
    rule2: "toto"
```

Ce fichier ne sera pas chargé, c'est juste un modèle. Il faut ensuite, dans la doc d'installation du bundle, demander de
copier/coller le fichier dans config/packages et définir sa configuration. (Vous remarquerez que le chemin dans le
bundle est le même que dans l'application
Symfony).

## Gestion des services

- On a besoin du fichier `config/services.yaml`.
- Dans `EntrepriseSkeletonBundle.php`, importer le fichier (Pour le coup, ce fichier est bien "chargé" automatiquement par
  Symfony).

```php
    public function loadExtension(array $config, ContainerConfigurator $containerConfigurator, ContainerBuilder $containerBuilder): void
    {
        $containerConfigurator->import('../config/services.yaml');
    }
```

- Définir vos services dans `config/services.yaml`.

## Gestion des contrôleurs/routes.
Dans config/routes/entreprise_skeleton.yaml
```yml
amu_skeleton:
  resource:
    path: '@EntrepriseSkeletonBundle/src/Controller/'
    namespace: Entreprise\SkeletonBundle\Controller
  type: attribute
  prefix: entreprise_skeleton
```
- Un controller est un service. Mettre en place la gestion des services.
- La gestion des routes n'est pas automatique. Le fichier dans `config/routes/` n'est pas chargé automatiquement. Il
  faut donc soit mettre en place une recette, mais c'est contraignant, soit vous mettez dans la doc d'installation qu'il
  faut copier/coller le fichier. (Vous remarquerez que le chemin dans le bundle est le même que dans l'application
  Symfony).
