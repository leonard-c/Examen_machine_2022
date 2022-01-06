# Examen_machine_2022
Examen machine du 6 Janvier 2022 : compression d'image parallèle par série de Fourier

**Dans le Makefile, modifier les premières lignes en commentant ou décommentant pour prendre le fichier adapter à votre environnement !**

__Pour ceux en distanciel__ : Pour toute question, envoyer un mail à xjuvigny@gmail.com (plus rapide que juvigny@onera.fr)


# Rapport :

1. Mesure de temps :

On mesure le temps pris par les différentes étapes du programme.

• Avec tiny_lena_gray.png

Étape                       		   	| Temps (s)
----------------------------------------|-----------
Encodage de l’image par DFT 	       	| 0.814636
Sélection des p% plus gros coeﬀicients	| 0.00024
Reconstitution de l’image ”compressée” 	| 0.086561

• Avec small_lena_gray.png

Étape                       		   	| Temps (s)
----------------------------------------|-----------
Encodage de l’image par DFT 	       	| 214.129
Sélection des p% plus gros coeﬀicients	| 0.005505
Reconstitution de l’image ”compressée” 	| 21.7543


2. Parallélisation OpenMP

L'encodage de l'image par DFT  et la reconstitution de l'image compressée étant les fonctions qui prennent le plus de temps, on parallélise ces parties.
La parallélisation de la DFT se fait sur les lignes de X (résultat de la transformation de Fourrier), donc sur des féquences.
La parallélisation de la reconstitution se fait quant à elle sur les lignes de x (image compressée), donc sur des pixels.

On obtient les temps et accélérations :

Image                | Fonction 		| Nb de thread | Temps    | Accélération
---------------------|------------------|--------------|----------|--------------
tiny_lena_gray.png   | DFT      		| 1            | 0.836474 | 1
/                    | /                | 2            | 0.419005 | 2.00
/                    | /                | 3            | 0.278571 | 3.00
/                    | /                | 4            | 0.207857 | 4.02
/                    | /                | 8            | 0.116916 | 7.15
/                    | Reconstitution   | 1            | 0.083532 | 1
/                    | /                | 2            | 0.040613 | 2.06
/                    | /                | 3            | 0.028749 | 2.91
/                    | /                | 4            | 0.020349 | 4.10
/                    | /                | 8            | 0.011972 | 6.98
small_lena_gray.png  | DFT      		| 8            | 30.9804  | 6.91
/                    | Reconstitution   | 8            | 3.09225  | 7.04

NB: on a considéré uniquement les étapes parallélisées, car ce sont les seules qui changent.

On observe une importante accélération de ces étapes avec le nombre de processus. Cependant, on remarque que cette accélération n'est pas optimale pour un grand nombre de processus (notammement 8). Ce ralentissement peut être du à la petite partie en séquenciel (création et premier remplissage du tableau au début des fonctions) et peut-être à la granularité.

3. Première parallélisation MPI

On parallélise la DFT sur les lignes de l'image, puis on regroupe tout avec un MPI_Gather.

On obtient :

Image                | Nb de processus | Temps (DFT) | Accélération
---------------------|-----------------|-------------|--------------
tiny_lena_gray.png   | 1        	   | 211.943     | 1
/                    | 2               | 52.0559     | 4.07
/                    | 3               | 23.0782     | 9.18
/                    | 4               | 13.0547     | 16.23
small_lena_gray.png  | 1        	   | 0.821715    | 1
/                    | 2               | 0.199926    | 4.11
/                    | 3               | 0.084564    | 9.72
/                    | 4               | 0.048458    | 16.96

NB : Ici on ne considère que le temps pris par la fonction DFT car c'est le seul qui change.

