<h1 style="text-align: center;" markdown="1">TAL: Classification de documents par rapport à leur type de discours à l'aide de l'analyse discursive</h1>


<h3 style="text-align: right;" markdown="1">René Traoré, de Bézenac Emmanuel</h3>



## Introduction

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;L’analyse du discours dans le contexte du traitement automatique du langage a pour but une compréhension plus fine et plus en profondeur du texte en tentant de dégager une structure discursive dans celui-ci. Certaines subtilités sont difficilement détectables avec la plupart des modèles utilisés en traitement automatique du langage, qui se basent essentiellement sur des traits surfaciques comme le nombre d’occurrences d’un mot dans un document par exemple. Prenons un exemple qui illustre l’importance de pouvoir exploiter la structure discursive d’un texte trouvé dans [NLPERS]:

    1: J’aime uniquement voyager en Europe. Alors j’ai soumis un article à ACL.
    2. J’aime uniquement voyager en Europe. Néanmoins j’ai soumis un article à ACL. 

<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Lorsque ces deux couples de phrases sont traités indépendemment, on peut inférer les mêmes connaissances: Il aime voyager en Europe, et il a soumis un article à ACL. Mais grâce à l’analyse du discours nous pouvons inférer davantage d’informations, à savoir, en 1: ACL se déroule en Europe, et en 2:  ACL n’est pas en Europe. </br>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Plus concrètement, l’analyse discursive vise d’abord à segmenter le texte en unités élémentaire du discours (EDU),  puis à les rattacher, et identifier leur type de relation, puis récursivement les paires attachées sont liées à des segments simples ou complexes pour aboutir à une structure couvrant le document. Voici un exemple donné par [Braud, Denis 2013]:

    Exemple 1.1: {{[La hulotte est un rapace nocturne] [mais elle peut vivre le jour.]} contrast [La hulotte mesure une quarantaine de centimètres.]} continuation

    Exemple 1.2: {[Juliette est tombée.] [Marion l’a poussée.]} explanation
    
    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Le texte entre crochets représente un EDU, tandis que les accolades représentent l’existence d’une relation entre la paire de segments contenue par les accolades. Notons qu’il est possible d’avoir des relations entre EDU, mais également entre paires d’EDU, et ceci récursivement.

&nbsp;


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Un type de discours est une forme de langage utilisé par l’émetteur (énonciateur) en fonction du
but qu’il vise et du message qu’il souhaite transmettre à ses lecteurs. L’émetteur pourrait
vouloir :
* Raconter une histoire : dans ce cas, il utiliserait le **discours narratif**
* Expliquer un fait ou donner une information : il utiliserait le **discours explicatif** 
* Essayer de convaincre ou de faire changer d’opinion à ses lecteurs sur un sujet précis : **discours argumentatif**



&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Il pourrait être intéressant de voir si une corrélation entre analyse du discours et type de discours existe, et si oui, comment elle se traduit. Ce projet à pour but d'étudier cette question, en construisant des représentations latentes de documents  basées sur l'analyse du discours pour tenter de prédire le type de discours associé à celui-ci.



## I - Données - Constitution du corpus

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Faute d'avoir trouvé un corpus étiqueté préexistant pour notre tache de classification, nous avons du en consituer un. Nous avons dû nous poser quelques hypothèses simplificatrices pour pouvoir récupérer des documents en taille et en nombre suffisant. Nous savons que plusieurs types de discours peuvent être présents dans un seul texte au même moment. Nous faisons l'hypothèse qu'il n'y a qu'un seul pour un document. Nous supposons également que les documents intra-classe ne sont pas extraits de sources hétérogènes, sont représentatifs de la classe à laquelle ils appartiennent.




* **narratif**: constitué de la partie "Plot" dans des pages de films sur wikipedia.org.
* **argumentatif**: constitué des en-têtes des pages accessibles depuis la page "Science" de wikipedia.org. 
* **informatif**: consitué des dicours de présidents des États-Unis, accessibles depuis millercenter.org.

| Type de dicours    |     Taille moyenne (mots) |   Nombre     |  Origine |
| :-------------      | :---------------: | :---------:  | :---------: |
| **narratif**       |      694          |   731         | wikipedia.org|
| **argumentatif**   |      935          |   192         | wikipedia.org|
| **informatif**     |      400          |   226         | millercenter.org|


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Nous pouvons voir que les données inter-classe sont très hétérogènes. Il faudra en tenir compte durant la procédure d'évaluation, afin de ne pas biaiser la prédiction. Pour les données extraites depuis wikipedia.org, nous avons utilisé une API *wikipedia* pour Python. Pour les données de type argumentatives, il a fallu créer un webscrapper à l'aide de *BeautifulSoup*, qui extrait des liens à partir de la page principale, pour ensuite récupérer et filtrer l'information contenue sur la page hmtl.


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Nous avons ensuite effectué un pre-processing sur ces données: élimination des caractères inconnus, et application d'une expression régulière afin de segmenter le texte en phrases.


## II - Du texte à la représentation



&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Pour appliquer nos modèles de classification, il faut pouvoir représenter un document, en le projetant dans un espace latent. Une méthode très utilisée dans le traitement automatique du langage est de projeter les documents dans un espace caractérisé par le vocabulaire utilisé. Cette technique, dite *Bag of Words* peut s'avérer très utile pour certains cas de figure, mais on lui reproche souvent de ne pas tenir compte de l'ordre des mots dans le document, et de ne pas pouvoir capter la sémantique de ce dernier. Le vocabulaire utilisé dans un texte d'un type de discours particulier semble ne pas le caractériser suffisamment, même si l'on pourrait s'attendre à pouvoir extraire quelques informations pertinentes, des traits grammaticaux discriminants pour un type de texte particulier. Il pourrait donc être intéressant d'utiliser l'analyse du discours qui fait une analyse plus "profonde" du document pour trouver des traits plus pertinents et plus représentatifs de la classe des documents, mais aussi et surtout de la distinction entre ces derniers.


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Nous avons donc choisi d'utiliser un parseur *DPLP* *Representation Learning for Text-level Discourse Parsing*, de **[Ji Eisenstein 2014]**, étant l'état de l'art en terme de parseur discursif (disponible depuis https://github.com/jiyfeng/DPLP). reconstitue la structure de dépendances des éléments de discours élémentaire, avec ainsi que les relations associés, structure qui se traduit sous forme d'arbre binaire.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Mais avant de pouvoir utiliser l'utiliser, il faut appliquer des pré-traitements aux documents:
    * POS-tagging
    * lemmatisation
    * Named Entity Recognizer
Pour cela, nous avons utilisé CoreNLP de Stanford, une suite d'outils pour le traitement automatique de la langue, accessible depuis le lien suivant: http://stanfordnlp.github.io/CoreNLP.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Une fois CoreNLP et DPLP appliqués sur notre corpus, nous avons une structure arborescente par document. Nous avons modifié l'algorithme DPLP pour pouvoir renvoyer des arbres au format *nltk.Tree*. Voici une visualisation d'un arbre après traitements:

### Test



![arbre](images/arbre.png)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Comme avec les *Bags of Words*, nous avons besoin d'une représentation à donner à nos classifieurs. La première représentation choisie reprend le concept de la représentation *Bag of Words*, mais au lieu de prendre le vocabulaire pour caractériser notre espace, nous utilisons les différentes relations présentes dans un document. Voici un exemple plus concret:

|Repr. du doc n°i    | NS-elaboration |   SN-temporal    |  SN-purpose |  NN-textualorg | NS-purpose |
| :-------------      | :---------------: | :---------:  | :---------: |:---------: |:---------: |
| **bin**      |      0          |   1         | 1|  0 |  0 |
| **count**   |      0          |   4         | 6|  0 |  0 |
| **count_norm**     |      0          |   0.4   | 0.6|  0 |  0 |
| **tfid**     |      0          |   0.7         | 0.3|  0 |  0 |
| **count_height**     |    0     |   10   | 40  | 0  | 0  |
| **count_height_norm**     |    0     |   0.1   | 0.4 | 0  | 0  |


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Nous sommes donc passés d'une représentation arborescente d'un texte à une représentation vectorielle. Le vecteur **bin** contient un 1 sur la composante i si la relation est présente dans le document, et 0 sinon. Le vecteur **count** compte le nombre d'occurences des différentes relations. Le vecteur **count_norm** est la version normalisée pour que la somme des composantes soit égale à 1. La composante i du vecteur **count_norm** est donc la fréquence d'apparition de la relation associée à la composante i. De cette manière, nous avons une représentation qui est indépendante de la taille du document. La composante i du vecteur **count_height** est la somme de la hauteur de tout les noeuds qui possèdent la relation associée à la composante i. Tout ces vecteurs sont de taille 41, comme il y a 41 relations différentes présentes dans le corpus.


### Features additionnels

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Afin d'améliorer notre classifieur, nous pouvons ajouter des traits à nos représentations. Nous avons choisi d'ajouter une représentation *Bags of Words* des différent POS-Tags associée aux documents. Nous avons donc comme trait additionnel, une représentation fréquentielle du nombre de POS-Tags dans un document. Chaque vecteur à une taille de 36; nous avons 36 POS-tags présent dans le corpus. Le vecteur associé au document i est alors concaténé au vecteur i calculé précédemment. Il peut être utile de multiplions chaque composant du nouveau vecteur dans le but de contrôler son importance dans la classification. Notre nouvelle représentation du document est donc de taille 77. 



## III -  Noyaux

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Une fois nos représentations de vectorielles de documents calculés, il nous faut maintenant définir des métriques adaptés pour pouvoir calculer des similarités (ou des dissimilarités) entre nos vecteurs. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Pour calculer des similarités, nous avons eu recours à plusieurs noyaux:


* **Noyau linéaire**: $\large\quad k(x,y) = x^T.y \quad    \normalsize x,y \in \mathbb{R}^n$


* **Noyau similarité cosinus**: $\large\quad k(x,y) = \frac{x^T.y}{\left \|x\right \|.\left \|y\right \|} \quad    \normalsize x,y \in \mathbb{R}^n$


* **Noyau RBF**: $\large \quad k(x,y) = e^{\left \|x - y  \right \|^2} \quad \normalsize x,y \in \mathbb{R}^n$


* **Noyau tree Kernel**: Les représentations vectorielles citées plus haut semblent perdre une information importante: ils ne tiennent pas compte de la position relative des noeuds (et donc des positions relatives des relations) dans l'arbre généré par l'analyse discursive. Nous avons donc implémenté notre version de treeKernel **[A. Moschitti. 2006]**, qui, étant donné 2 arbres, calcule le nombre de sous-arbres en commun. Nous avons fait un *pruning* de l'arbre pour ne garder que les branches les plus proches de la racine (une distance de k noeuds de la racine), pour limiter le nombre de calculs (le calcul d'un tree Kernel est linéaire en le nombre de noeuds, mais le nombre de noeuds est exponentiel). Ceci n'est pas dérangeant: nous faisons la supposition que les relations associés aux noeuds plus proches de la racine représentent mieux le document.


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Pour calculer des dissimularités, nous avons utilisés les distances suivantes:

* **distance euclidienne** : $\large \quad d(x,y) = \left \|x - y  \right \|^2 \quad \normalsize x,y \in \mathbb{R}^n$


* **distance de minkowski, avec p=3** : $\large \quad k(x,y) = \left \|x - y  \right \|_p \quad \normalsize x,y \in \mathbb{R}^n$

Pour toutes nos représentations vectorielles, nous avons calculés les matrices de Gram $G$, définient comme ci-dessous: $\large \quad G_{i,j} = k(x_i,y_j) \quad \normalsize G \in \mathbb{R}^{n \times n}$

Nous connaissons maintenant les similarités entre chaque document, et pour chaque représentation. Nous pouvons ensuite appliquer des modèles de classification.


## IV - Modèles


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Nous avons utilisé des modèles de classification de la librairie *sklearn*. Voici une liste de ceux que nous avons utilisés:

* **Machines à vecteurs support**
* **k-plus proches voisins**
* **Random Forest**
* **Maxent**



Voici un récapitulatif du "pipeline" mis en place:

<img src="images/pipeline.jpg" style="width: 900px"/>


## V - Résultats, Interprétations

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Durant toute l'expérience, l'évaluation des modèles et des représentations est basée sur le taux de bonne prédiction des modèles. Les modèles sont tous évalués sur des données d'entrainement et de test séparés, en cross-validation (en séparent le corpus en blocs de 5 parties distinctes, tirées aléatoirement). Le sigle <code>(+/- var)</code> signifie qu'il y a une variance empirique de <code>var</code> sur les 5 tests successifs.

### Tree Kernel

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Nous avons évalués notre classifieur (SVM, avec C=1), en utilisant le noyau *Tree Kernel*. Avant de calculer celui-ci, nous avons effectué un *pruning* de l'arbre; en ne gardant que les noeuds avec une distance de 4 et 5 du noeud racine. Les résultats ne sont pas très intéressants, mais restent malgré tout meilleurs que la classification aléatoire (0.33, comme nous avons 3 classes). Les arbres sont de profondeur moyenne de 35, nous n'aurions pu effectuer le calcul des *treeKernels sans le *pruning*. L'arbre "raccourci" semblent ne pas capter assez d'informations pour prédire le label correctement. 

<table border=\1\ class=\dataframe\>
  <thead>
    <tr style=\text-align: right;\>
      <th></th>
      <th>pruning size: 4</th>
      <th>pruning size: 5</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>treeKernel</th>
      <td>0.51   (+/- 0.09)</td>
      <td>0.53   (+/- 0.08)</td>
    </tr>
  </tbody>
</table>


### Représentations vectorielles

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Nous passons ensuite aux représentations des documents sous la forme vectorielle. Voici les résultats, appliqués sur les modèles de classification énoncés plus haut.



#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; K-plus proches voisins
<table border=\1\ class=\dataframe\>
  <thead>
    <tr style=\text-align: right;\>
      <th></th>
      <th>bin</th>
      <th>count</th>
      <th>height</th>
      <th>norm</th>
      <th>tfid</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>euclidean distance</th>
      <td>0.69   (+/- 0.13)</td>
      <td>0.71   (+/- 0.06)</td>
      <td>0.67   (+/- 0.03)</td>
      <td>0.62   (+/- 0.07)</td>
      <td>0.65   (+/- 0.06)</td>
    </tr>
    <tr>
      <th>minkowski distance</th>
      <td>0.69   (+/- 0.13)</td>
      <td>0.71   (+/- 0.06)</td>
      <td>0.67   (+/- 0.03)</td>
      <td>0.62   (+/- 0.07)</td>
      <td>0.65   (+/- 0.06)</td>
    </tr>
  </tbody>
</table>


#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Random Forest

<img src="images/random_for_res.png" style="width: 400px"/>


<table border=\1\ class=\dataframe\>
  <thead>
    <tr style=\text-align: right;\>
      <th></th>
      <th>bin</th>
      <th>count</th>
      <th>height</th>
      <th>norm</th>
      <th>tfid</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>crit = gini</th>
      <td>0.68   (+/- 0.04)</td>
      <td>0.76   (+/- 0.08)</td>
      <td>0.78   (+/- 0.08)</td>
      <td>0.76   (+/- 0.06)</td>
      <td>0.74   (+/- 0.02)</td>
    </tr>
  </tbody>
</table>


#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Machines à vecteur support

<img src="images/svm_res.png" style="width: 400px"/>


<table border=\1\ class=\dataframe\>
  <thead>
    <tr style=\text-align: right;\>
      <th></th>
      <th>bin</th>
      <th>count</th>
      <th>height</th>
      <th>norm</th>
      <th>tfid</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>rbf</th>
      <td>0.72   (+/- 0.03)</td>
      <td>0.69   (+/- 0.07)</td>
      <td>0.40   (+/- 0.05)</td>
      <td>0.35   (+/- 0.00)</td>
      <td>0.38   (+/- 0.07)</td>
    </tr>
    <tr>
      <th>lin</th>
      <td>0.72   (+/- 0.04)</td>
      <td>0.80   (+/- 0.03)</td>
      <td>0.79   (+/- 0.05)</td>
      <td>0.57   (+/- 0.05)</td>
      <td>0.63   (+/- 0.03)</td>
    </tr>
    <tr>
      <th>cos_sim</th>
      <td>0.70   (+/- 0.03)</td>
      <td>0.60   (+/- 0.09)</td>
      <td>0.53   (+/- 0.03)</td>
      <td>0.60   (+/- 0.09)</td>
      <td>0.69   (+/- 0.02)</td>
    </tr>
  </tbody>
</table>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;En regardant les coefficients associés aux composants des vecteurs, nous avons pu extraire les 4 relations qui participent à la discimination entre les classes.

* **narratif**: NN-temporal, NS-concession, NS-example, NS-antithesis,

* **argumentatif**: SN-attribution, NN-sequence, NN-list, SN-antithesis,

* **informatif**: SN-condition, NS-condition, NS-manner, NN-list.


#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Maxent


<table border=\1\ class=\dataframe\>
  <thead>
    <tr style=\text-align: right;\>
      <th></th>
      <th>bin</th>
      <th>count</th>
      <th>height</th>
      <th>norm</th>
      <th>tfid</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>maxEnt</th>
      <td>0.74   (+/- 0.07)</td>
      <td>0.80   (+/- 0.06)</td>
      <td>0.79   (+/- 0.06)</td>
      <td>0.59   (+/- 0.05)</td>
      <td>0.66   (+/- 0.07)</td>
    </tr>
  </tbody>
</table>
<img src="images/maxent_1.png" style="width: 400px"/>


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Nous avons des résultats intéressants: Nous avons jusqu'à 80% de bonne classification avec une représentation vectorielle *count* (comptage du nombre de relations présents dans tout l'arbre) avec les modèles *SVM* et *MaxEnt*. Ce résultat est potentiellement biaisé car comme vu précedemment, la taille de nos documents varient conséquemment en fonction de nos classes, et donc le choix discriminant du modèle pourrait être effectué sur la taille des documents du corpus (la taille d'un texte ne reflète pas son type de discours), plutôt que sur l'information qui caractérise bien un type de discours. D'autres représentations vectorielles (telles que *bin*, *norm*, ou *tfid*) semblent être plus robustes à la variance de la taille intra-classe d'un texte (et donc de la profondeur et du nombre de noeuds associés à celui-ci). Nous avons cependant de plus faibles scores avec ces derniers: 74% pour *bin* et *MaxEnt*, et 69% pour *tfid* et *SVM*.

<h4 style="text-align: center;" markdown="1">Aperçu de la Matrice de Confusion, pour le classifieur *MaxEnt* appliqué sur *count*</h4>

<img src="images/conf_maxent.png" style="width: 400px"/>

<h3 style="text-align: left;" markdown="1">Ajout de features supplémentaires</h3>



&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Afin d'affiner notre score, nous tentons d'ajouter d'autres features, qui pourraient capter des traits grammaticaux potentiellement représentatifs d'une classe. Nous avons d'abord essayé d'ajouter une combinaison linéaire de deux Noyaux: le noyau similarité linéaire avec le noyau *Tree Kernel*. Nous avons appliqué un *SVM*, mais résultats étaient encore plus faibles qu'en prenant le noyau de similarité seul.

Nous nous sommes ensuite tournés vers l'étiquetage morpho-syntaxique: nous avons, comme précédemment, extrait des vecteurs représentant la distribution des POS-Tags dans un document (de type *Bag of Words*). Nous les avons concaténés avec les anciennes représentations, et à nouveau, appliqué les classifieurs:

<h4 style="text-align: left;" markdown="1">Maxent + POS-Tags</h4>


<table border=\1\ class=\dataframe\>
  <thead>
    <tr style=\text-align: right;\>
      <th></th>
      <th>bin</th>
      <th>count</th>
      <th>height</th>
      <th>norm</th>
      <th>tfid</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>maxent+</th>
      <td>0.94   (+/- 0.04)</td>
      <td>0.96   (+/- 0.02)</td>
      <td>0.95   (+/- 0.02)</td>
      <td>0.97   (+/- 0.03)</td>
      <td>0.97   (+/- 0.03)</td>
    </tr>
  </tbody>
</table>


<img src="images/maxentplus.png" style="width: 400px"/>


<h4 style="text-align: left;" markdown="1">SVM + POS-Tags</h4>

<table border=\1\ class=\dataframe\>
  <thead>
    <tr style=\text-align: right;\>
      <th></th>
      <th>bin</th>
      <th>count</th>
      <th>height</th>
      <th>norm</th>
      <th>tfid</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>linear</th>
      <td>0.98   (+/- 0.02)</td>
      <td>0.97   (+/- 0.01)</td>
      <td>0.96   (+/- 0.03)</td>
      <td>0.99   (+/- 0.03)</td>
      <td>0.99   (+/- 0.02)</td>
    </tr>
  </tbody>
</table>
<img src="images/svmresplus.png" style="width: 400px"/>




&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; L'erreur de classification des documents est très faible. Nous avons donc reproduit notre expérience, en se basant uniquement sur les traits morphosyntaxiques: nous avons eu un score de 93% de bonne classification. L'essentiel de l'information captée semble être contenue dans la représentation *Bag of Words* des POS-Tags, même si l'analyse de la structure discursive semble avoir apportée une information additionnelle, et complémentaire sur la distinction entre types de documents.




<h4 style="text-align: center;" markdown="1">Analyse en composantes principales des documents du corpus</h4>
<img src="images/acp.png" style="width: 800px"/>


<h4 style="text-align: center;" markdown="1">TSNE des documents du corpus</h4>
<img src="images/tnse.png" style="width: 800px"/>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ces graphiques semblent attester des très bon résultats obtenus par nos classifieurs: nous avons bien trouvé un espace caractérisant des documents capable de séparer les différentes classes.

## Conclusion

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;L'information résultant de l'analyse du discours, ainsi que de l'analyse morpho-syntaxique semble être suffisante pour décrire les subtilités et distinctions entre classes du corpus. Néanmoins, il faudrait pouvoir tester les classifieurs appris sur le corpus sur des données provenant d'autres sources pour être sûr que l'information captée par ceux-ci exploitent bien les distinctions entre types de discours.

## Bibliographie

* **[A. Moschitti. 2006]**: Making tree kernels practical for natural language processing. In Conference of the European Chapter of the Association for Computational Linguistics (EACL), 2006a.


* **[Ji Eisenstein 2014]**: Representation Learning for Text-level Discourse Parsing, 2014, Yangfeng Ji, School of Interactive Computing , Georgia Institute of Technology INRIA, and Jacob Eisenstein, School of Interactive Computing, Georgia Institute of Technology.
