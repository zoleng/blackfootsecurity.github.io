# Blackfoot Security Blog

Ce répo permet de générer le site https://blackfootsecurity.github.io/. Il utilise Jekyll avec le thème jasper2.

## Comment ça marche

La branche `develop` contient les sources qui génèrent le blog. Toutes les modifications au site doivent être faites sur `develop`, éditer master ne servirait absolument à rien.

Vous pouvez fork le répo et modifier votre fork autant que vous voulez sans jamais prendre le risque d'affecter le site en prod. Quand vous voulez publier un article, faites une Pull Request. Comme le site blackfootsecurity reste un blog professionel, le contenu que vous postez dans la PR sera quand même vérifié à avant d'être merge.

## Déployer le blog localement

Pour vérifier vos changements vous pouvez générer une version locale du site. Les étapes faire ça sont:

1. Installer Ruby `>=2.1.0` si vous ne l'avez pas
2. Installer Bundler: `gem install bundler`
3. Cloner votre fork du répo
4. Entrez dans le repo et assurez vous bien que vous êtes sur la branche `develop`
5. `bundle install`
6. `bundle exec jekyll serve`

Le blog est maintenant compilé, vous pouvez y accéder via http://127.0.0.1:4000/

## Écrire un article

Les articles du blog vont dans `_posts/`, et doivent être nommés `YYYY-MM-DD-titre.md`, ou `YYYY-MM-DD` est la date d'ecriture de l'article. Vous pouvez vous baser sur un des posts déjà présents pour voir comment formatter le contenu, il y a aussi la [doc jekyll](https://jekyllrb.com/docs/posts/).

Vous pouvez écrire en Markup ou en HTML directement, comme vous préférez, il y a plusieurs exemples dans la documentation Jekyll. Pour un exemple de page en HTML, le thème qu'on utilise a une bonne [démo](https://jekyller.github.io/jasper2/a-full-and-comprehensive-style-test) ([code source](https://github.com/jekyller/jasper2/blob/master/_posts/2012-09-01-a-full-and-comprehensive-style-test.html)).

## La page auteur

Chaque membre de l'équipe peut mettre plus ou moins ce qu'il/elle veut sur sa page auteur, en gardant en tête que ça reste tout de même une page professionelle.

Vous pouvez vous ajouter et éditer votre page auteur dans le fichier `_data/authors.yml`

## Tags

Si vous voulez utiliser un tag qui n'existe pas encore, vous pouvez le créer dans `_data/tags.yml`. Une petite note, le champ `name` doit être le nom du tag en minuscule, vous pouvez utiliser `displayname` pour changer comment ce tag est affiché sur sa page dédiée `/tag/{name}`. Les tags, comme la plupart des éléments du site, peuvent contenir un champ `cover` qui permet d'avoir une bannière personnalisée.
