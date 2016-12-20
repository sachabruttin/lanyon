---
layout: post
title: Gérer les permissions de contenu dans Orchard
tags:
 - Orchard CMS
redirect_from:
 - /gerer-les-permissions-de-contenu-dans-orchard
---

Parfois, il peut-être nécessaire de gérer des permissions sur le contenu de son site Orchard.
Depuis quelques version, un module existe et fait directement partie du code source. Son nom est "Content Item Permissions" et
il est décrit comme "Allows item-level front end view permissions.".

Malheureusement, la documentation de ce module est quasi inexistante ou alors je ne l'ai pas trouvée :confused:
En fait, il s’agit d’un Content Part qu’il faut donc ajouter à la définition du ou des Content Types que l’on veut pouvoir sécuriser.

Nous allons donc voir ici comment mettre en place ce module.
Premièrement, il faut activer le module...

![Activate Module](/public/images/2014-02-26-gerer-les-permissions-de-contenu-dans-orchard/activate_module_thumb.png)

Ensuite il faut ajouter le Content Part sur le Content Type choisis.
Pour l’exemple nous allons modifier le type Page. Allez dans "Content Definition" et cliquez sur le bouton "Edit" correspondant.

Cliquez ensuite sur le bouton "Add Parts" et sélectionnez "Content Permissions" dans la liste et validez avec "Save".
La page d'édition du Content Type s'affiche à nouveau et vous voyez maintenant la nouvelle part, cliquez sur le bouton d'expansion pour afficher
les paramètres par défaut.

![Content Permissions](/public/images/2014-02-26-gerer-les-permissions-de-contenu-dans-orchard/image_thumb_1.png)

Pour définir les droits d'une page, il faut maintenant éditer et sélectionner "Enable Content Item access control"...

Lorsqu’une page est des droits d'accès définis, un cadenas est affiché sur son item dans la liste "Content Items".

![Content Items](/public/images/2014-02-26-gerer-les-permissions-de-contenu-dans-orchard/image_thumb_2.png)
