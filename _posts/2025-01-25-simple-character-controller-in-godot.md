---
title: Création d'un *Character Controller* basique dans Godot
date: 2025-01-25
---

Dans la liste des éléments importants pour un jeu de plateforme, le **comportement** du personnage joueur est indubitablement en haut.<br>
En effet, les sensations procurées au joueur par ce dernier peuvent faire la différence entre un jeu *correct* et un *bon* jeu.

Aujourd'hui, nous verrons comment poser les **bases** d'un tel contrôleur, que vous serez libre de modifier à votre guise.

## Pré-requis. ##

Pour suivre correctement cet article, il est conseillé de comprendre les notions de variables et de fonctions.<br>
Une connaissance superficielle de l'architecture des noeuds et des inputs dans Godot est un plus.

## Mise en place de la scène "Player". ##

Nous allons pour cette démonstration utiliser une structure assez simple :

- **CharacterBody2D**

    > Ce type de noeud donne accès à des fonctions spécialisées pour les personnages dont les mouvements sont contrôlés par des scripts.

- **CollisionShape2D**

	> Ce type de noeud permet de donner une forme de collision à notre personnage (e.g. sphère, capsule...).

- **AnimatedSprite2D**

    > Ce type de noeud permet de lancer des animations depuis des feuilles de sprites.

Voici à quoi cela ressemble dans l'éditeur :

![Hiérarchie de la scène](/assets/images/controller-character/player_controller_1.png)
![AnimationPlayer](/assets/images/controller-character/player_controller_2.png)

Il ne reste plus qu'à attacher un script au **CharacterBody2D** et nous serons prêt à nous lancer.

![Ajout d'un script](/assets/images/controller-character/player_controller_3.png)

## Création du script "player.gd". ##

Rentrons maintenant dans le vif du sujet.

La première ligne du script est la suivante:

![extends CharacterBody2D](/assets/images/controller-character/player_controller_4.png)

Elle signifie que notre script est considéré comme un CharacterBody2D, et peut donc utiliser toutes les fonctions de ce type de noeud.

Nous allons commencer par créer ce que l'on appelle des **export variables**, c'est-à-dire des variables que l'on peut modifier directement dans l'éditeur, sans avoir à modifier systématiquement notre script.<br>
Pour ce faire, il suffit d'ajouter le mot-clé *@export* devant la déclaration de variable, comme suit :

![@export var gravity:int = 1200](/assets/images/controller-character/player_controller_5.png)

Nous avons donc ici créé une variable exportée de nom *gravity*, de type *int* et de valeur initiale *1200*.

Nous pouvons de même ajouter les variables dont nous aurons besoin par la suite :

![Liste des variables exportées](/assets/images/controller-character/player_controller_6.png)

> Note: le préfixe *@export_range(x, y)* permet de borner les valeurs entre x et y, et offre un slider pour la modification.

Dans Godot, la bonne pratique est de gérer tout ce qui concerne le **moteur physique** dans une fonction dédiée : *_on_physics_process(delta)*<br>
Ici, le paramètre *delta* correspond au temps écoulé depuis la dernière "image physique", qui peut être différente de la dernière image affichée à l'écran.<br>

En premier lieu, il nous faut récupérer la direction dans laquelle le joueur veut se déplacer.<br>
Pour ce faire, nous avons à notre dispositon le singleton **Input** et la fonction **get_axis**.<br>

![Input.get_axis()](/assets/images/controller-character/player_controller_7.png)

Nous recevrons donc dans la variable direction un chiffre compris entre *-1.0* et *1.0*, selon que le joueur appuie respectivement sur *gauche* ou *droite*.

Les prochaines lignes de code sont un peu plus compliquées, nous allons donc les voir une à une :

![Mouvement au sol](/assets/images/controller-character/player_controller_8.png)

> **velocity** correspond à un **vecteur** avec deux composantes : <br> *hozizontale (x)* et *verticale (y)*.<br>Nous ne modifierons ici que la composante x.

Nous vérifions d'abord avec un bloc **if** que le joueur a bien envoyé une commande mouvement, et donc que la variable *direction* n'est pas égale à *0*.<b>
Si c'est le cas, nous modifions la vélocité de notre personnage en conséquence.

> La fonction **lerp** (interpolation linéaire) permet de modifier une valeur de A à B, en fonction d'un facteur W allant de 0.0 à 1.0, comme suit : **lerp(A, B, W)**

Ensuite, si la variable *direction* est égale à *0*, le code contenu dans le bloc **else** sera exécuté et appliquera une interpolation en direction de *0.0*.

Les vitesses auxquelles le personnage accélère et ralentit sont définies par les variables *acceleration* et *friction*.

Enfin, comme nous ne voulons pas que le personnage accélère éternellement, nous limitons sa vitesse avec la fonction **clamp** :

![Clamp velocity](/assets/images/controller-character/player_controller_9.png)

> La fonction **clamp** consiste à borner une valeur entre un minimum et un maximum, comme suit : **clamp(valeur, minimum, maximum)**.

Si vous testiez le code actuellement, notre personnage ne bougerait pas et serait toujours orienté dans la même direction, nous allons donc remédier à ces deux points.

Pour commencer, nous allons ajouter une nouvelle variable exportée pour récupérer notre **AnimatedSprite2D** dans l'éditeur :

![@export var sprite:AnimatedSprite2D](/assets/images/controller-character/player_controller_10.png)
![Export dans l'éditeur](/assets/images/controller-character/player_controller_11.png)

Nous modifions alors la propriété **flip_h** de notre **AnimatedSprite2D**, qui permet d'appliquer un miroir horizontal, si notre vitesse est négative - c'est-à-dire que nous allons vers la gauche.<br>

> Nous utilisons ici une **expression ternaire** qui est une façon plus courte d'écrire un bloc **if/else** vu plus haut.

Pour terminer, nous allons appliquer la *velocity* calculée jusqu'ici avec la fonction **move_and_slide()** issue de **CharacterBody2D**, à laquelle nous avons accès grâce au mot-clé *extend* situé en haut du script.

![move_and_slide()](/assets/images/controller-character/player_controller_13.png)

Et notre personnage est maintenant capable de se déplacer de gauche à droite !

**Mais !** Si vous vouliez à nouveau tester le script, vous remarqueriez que notre personnage flotte dans les airs.<br>
En effet, nous n'avons toujours pas appliqué la *gravité*.<br>
Pour ce faire, nous allons utiliser une fonction bien pratique de **CharacterBody2D** qui nous permet de savoir si le personnage est placé sur une surface : **is_on_floor()**.

Nous pouvons donc appliquer la gravité ou non en fonction du résultat de cette fonction, avec un simple bloc **if/else**, avant d'utilise *move_and_slide*.

![Gravité](/assets/images/controller-character/player_controller_14.png)

> Il est possible de modifier les éléments avec lesquels le personnage intéragit en modifiant les *calques de collision*, mais ce sera le sujet d'un autre article.

Il est maintenant temps de rendre plus élégant notre mouvement en utilisant notre **AnimatedSprite2D** dans sa totalité.<br>
Pour ce faire, nous allons utiliser sa fonction **set_animation()** dans une nouvelle *expression ternaire*, en jouant l'animation *"run"* si le joueur a saisi une direction.

![Animation](/assets/images/controller-character/player_controller_15.png)

Et pourquoi pas ajouter le **saut** ?<br>
En effet, nous avons défini plus haut une variable *jump_speed* qu'il est temps d'utiliser.<br>
Avec tout ce que nous avons vu jusqu'à présent, il ne nous manque qu'une fonction située dans **Input** et qui nous permet de savoir si une touche vient d'être pressée : **is_action_just_pressed()**.<br>
En combinant cette fonction avec **is_on_floor()**, nous pouvons facilement créer une fonction de saut:

![Fonction de saut](/assets/images/controller-character/player_controller_16.png)

On peut ensuite ajouter une animation de saut dans notre **AnimatedSprite2D** et modifier notre gestion des animations pour la jouer si le personnage n'est pas sur le sol:

![Modification des animations](/assets/images/controller-character/player_controller_17.png)

Il est maintenant temps de voir le script dans son intégralité :

```
extends CharacterBody2D

@export var speed: int = 300
@export var max_speed: int = 3000
@export_range(0.0, 1.0) var acceleration: float = 0.32
@export_range(0.0, 1.0) var friction: float = 0.20
@export_range(0.0, 1.0) var air_friction: float = 0
@export var gravity: int = 1200
@export var fall_speed: int = 280
@export var jump_speed: int = -450

@export var sprite: AnimatedSprite2D

func _physics_process(delta) -> void:
	
	# Nous récuperons ici la direction donnée par le joueur
	var direction: float = Input.get_axis("ui_left", "ui_right")
	
	# Nous gérons les mouvements en fonction de la direction demandée
	if direction != 0:
		velocity.x = lerp(velocity.x, direction * speed, acceleration)
	else:
		velocity.x = lerp(velocity.x, 0.0, friction)
		
	velocity.x = clamp(velocity.x, -max_speed, max_speed)
	
	sprite.set_flip_h(true) if velocity.x < 0 else sprite.set_flip_h(false)
	
	# Nous appliquons la gravité
	if is_on_floor():
		velocity.y = 0
	else:
		velocity.y += gravity * delta
	
	if Input.is_action_just_pressed("jump") and is_on_floor():
		velocity.y = jump_speed
	
	move_and_slide()
	
	# Nous modifions les animations si nécessaire
	if !is_on_floor():
		sprite.set_animation("jump")
	else:
		sprite.set_animation("run") if direction != 0 else sprite.set_animation("idle")
```

Et voici le code en **action** !

![Action !](/assets/images/controller-character/character.gif)

Nous verrons comment complexifier ce type de contrôleur dans un autre article, où nous aborderons les **State Machines**.

En attendant, bon développement !

~Ryn