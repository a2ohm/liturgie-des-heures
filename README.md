# Notice explicative du fonctionnement interne

## A. Fonctionnement général
### 1. Squelette et variables de session
Tous les offices s’appuient sur un squelette dédié au format html. Chaque partie de l’office est insérée dans un `<div>` dont l’identifiant est unique. Vous trouverez en annexe les différentes parties pour les différents offices. Ces squelettes se trouvent à la racine de l’application et portent un nom explicite : `lectures.html`, `laudes.html`, `milieu.html`, `vepres.html`, `complies.html`.

Toutes les parties de l’office sont présentes dans des fichiers différents. Le programme détermine les bons fichiers en fonction du temps liturgique et du calendrier de célébration des saints. Le principe est simple : pour chaque `<div>`, il existe une variable de session (cf. `sessionStorage` dans le DOM) où l’on renseigne le chemin d’accès au fichier requis.

Ex : l’antienne du premier psaume de l’office des lectures est spécifiée dans la variable de session : `sessionStorage.l_antienne_1`. Sa valeur pour le premier dimanche de l’Avent est : `'lectures/antiennes/antienne_1_1_0.html'`.

### 2. Temporal
La première étape consiste à déterminer les parties de l’office (en renseignant donc les bonnes variables de session), en ne tenant compte que du temporal.

Il y a des temps liturgiques particuliers qui nécessitent un traitement spécifique : les féries du temps de l’Avent, le temps de Noël, la semaine sainte, l’octave de Pâques, l’Ascension et la Pentecôte. Ces temps, on vient charger un fichier temporal, qui fonctionne globalement comme les fichiers du sanctoral (cf. supra pour la description de ce fonctionnement).

### 3. Sanctoral
Après la détermination des éléments de l’office du temporal, si l’on célèbre un saint, on vient écraser les valeurs des variables de session à l’aide des parties obligatoires spécifiées au propre du saint. Pour ce faire, on charge un fichier : `'sanctoral/MM/JJ-office.html'`[^1] qui ne contient que du code javascript.

*Attention, en ce qui concerne la possibilité d’afficher le commun à partir du capitule pour les mémoires, cette option est gérée par une fonction spécifique, mais aucunement dans le fichier sanctoral.*

[^1]: MM = mois (01, 02...) ; JJ = jour du mois (01, 02 ..., 31) ; office = lectures / laudes / milieu / vepres.

Exemple de fichier sanctoral pour une mémoire (Saint Stanislas, 11/04) :

```javascript
var path = 'sanctoral/04/11-';
var propre = ['la_antienne_ev'];
sessionStorage.setItem('la_titre_sanctoral', path+'titre.html');

for (var i=0; i < propre.length; i++) {
    sessionStorage.setItem(propre[i], path+propre[i]+'.html');
}

sessionStorage.setItem('la_oraison', path+'oraison.html');
```

→ on modifie trois valeurs : le titre, l’oraison (toujours modifiés) et l’antienne évangélique dans ce cas précis.

Par convention, on utilise le même nom pour les fichiers à charger que pour les variables de session, avec le jour du mois comme préfixe et bien-sûr l’extention .html. Eg. pour la mémoire de S. Stanislas, le fichier de l’antienne évangélique des laudes, le fichier s’appelle : 11-la_antienne_ev.html. (voir le tableau en annexe pour la liste des parties de l’office).

Exemple de fichier sanctoral pour une fête (Saints Philippe et Jacques, 03/05) :

```javascript
var path_propre = 'sanctoral/05/03-';
var path_commun = 'sanctoral/apotres/';
var commun = [
    'la_antienne_inv',
    'la_hymne',
    'la_intercession'
];
var propre = [
    'la_antienne_1',
    'la_antienne_cantique',
    'la_antienne_2',
    'la_capitule',
    'la_repons',
    'la_antienne_ev'
];

sessionStorage.setItem('la_titre_sanctoral', path_propre+'titre.html');
sessionStorage.setItem('la_psaume_1', '62');
sessionStorage.setItem('la_cantique', '41');
sessionStorage.setItem('la_psaume_2', '149');

for (var i=0; i < propre.length; i++) {
    sessionStorage.setItem(propre[i], path_propre+propre[i]+'.html');
} 
for (var i=0; i < commun.length; i++) {
    sessionStorage.setItem(commun[i], path_commun+commun[i]+'.html');
} 
sessionStorage.setItem('la_oraison', path_propre+'oraison.html');
```
→ dans cet exemple, il y a plus de changements : toutes les parties sont modifiés, mais certaines sont prises au propre et d’autres au commun.

Noter la manière de changer la valeur d’un psaume : `sessionStorage.setItem('la_psaume_1', '62')`.

## B. Cas particuliers 
### 1. Paramètres
Les paramètres de l’application sont gérées dans des variables locales (`localStorage` dans le DOM). Ce qui nous intéresse ici, ce sont propres régionaux.

### 2. Propres régionaux
Les valeurs possibles des propres régionaux sont les suivantes : `'afriquen'`, `'belgique'`, `'canada'`, `'europe'`, `'france'`, `'luxembourg'`, `'suisse'`.

La gestion des particularités régionales est assurée par le fonction Sanctoral_propre() du fichier lh-calendrier.js. Dans une telle situation, le programme cherchera non pas le fichier `'sanctoral/MM/JJ-office.html'`, mais le fichier : `'sanctoral/MM/JJ-office_region.html'`, où region vaut une des valeurs spécifiées ci-dessus.

### 3. Propre des diocèses
Les propres diocésains sont gérés de la même manière que les propres régionaux, la seule différence est qu’il ne peut y avoir qu’un seul diocèse sélectionné. Pour ajouter, les offices d’un diocèse, il suffit donc de me transmettre les fichiers suivants :
```
        JJ-office_diocese.html
        JJ-titre_diocese.html
        ... les fichiers spécifiques au diocèse.
```

Si un office n’est pas modifié (eg. milieu du jour pour une mémoire), dans ce cas, on ne met pas le fichier de l’office (cela générera un warning dans la console javascript, car on essaiera de charger le fichier quand même, mais cette erreur est sans incidence pour le fonctionnement du programme).

# Annexe

## A. Liste des sections pour chaque office 

### 1. Office des lectures
| Id de l’élément `<div>` = variable de session | Commentaire |
|---|---|
| `l_titre_principal` | Renseigné automatiquement par `lh.js` (date du jour) |
| `l_titre_sanctoral` | @ partir du fichier : `JJ-titre.html` |
| `l_titre_secondaire` | Renseigné automatiquement, contient les informations : temps liturgique, année liturgie, semaine, semaine du psautier. |
| `l_temporal` | Pour le temps particulier comme les féries du 17 au 24 décembre, l’octave de Noël ... permet le chargement d’un fichier qui personnalise les offices pour ces moments particuliers. |
| `l_sanctoral` | Invisible, permet de charger le fichier `JJ-lectures.html` qui permet d’insérer le propre du saint. |
| `l_introduction` | Automatique : *Dieu vient à mon aide* ... |
| `l_select-hymne-div` | Permet de sélectionner les offices quand plusieurs sont proposés. |
| `l_hymne` | Cette variable contient : soit l’url d’un fichier `html` qui contient le texte de l’hymne, soit le nom du fichier qui contient la liste des hymnes à choisir (liste qui permet de renseigner l’élément select de `l_select-hymne-div`) |
| `l_antienne_1` |  |
| `l_psaume_1` |  |
| `l_antienne_1b` | Reprend la valeur de l_antienne_1 si on répète / pas de variable de session |
| `l_antienne_2` |  |
| `l_psaume_2` |  |
| `l_antienne_2b` | Reprend la valeur de l_antienne_2... |
| `l_antienne3` |  |
| `l_psaume_3` |  |
| `l_antienne_3b` | Reprend la valeur de l_antienne_3... |
| `l_verset` |  |
| `l_lecture` |  |
| `l_patristique` |  |
| `l_tedeum` | n’est affiché que s’il s’agit d’une fête ou solennité. |
| `l_oraison` | |

### 2. Laudes
| Id de l’élément `<div>` | Commentaire |
|---|---|
| `la_titre_principal` | Renseigné automatiquement par `lh.js` (date du jour) |
| `la_titre_sanctoral` | @ partir du fichier : `JJ-titre.html` |
| `la_titre_secondaire` | Renseigné automatiquement, contient les informations : temps liturgique, année liturgie, semaine, semaine du psautier. |
| `la_temporal` | Pour le temps particulier comme les féries du 17 au 24 décembre, l’octave de Noël ... permet le chargement d’un fichier qui personnalise les offices pour ces moments particuliers. |
| `la_sanctoral` | Invisible, permet de charger le fichier `JJ-laudes.html` qui permet d’insérer le propre du saint. |
| `la_introduction` | Automatique : *Dieu vient à mon aide*... |
| `la_antienne_inv` |  |
| `la_select_inv` | Permet de sélectionner le psaume invitatoire |
| `la_select-hymne-div` | Permet de sélectionner les offices quand plusieurs sont proposés. |
| `la_hymne` | Cette variable contient : soit l’url d’un fichier `html` qui contient le texte de l’hymne, soit le nom du fichier qui contient la liste des hymnes à choisir (liste qui permet de renseigner l’élément `select` de `la_select-hymne-div`) |
| `la_antienne_1` |  |
| `la_psaume_1` |  |
| `la_antienne_1b` | Reprend la valeur de `la_antienne_1` / pas de variable de session |
| `la_antienne_cantique` |  |
| `la_cantique` |  |
| `la_antienne_cantiqueb` | Reprend la valeur de `la_antienne_cantique`... |
| `la_antienne_2` |  |
| `la_psaume_2` |  |
| `la_antienne_2b` | Reprend la valeur de `la_antienne_2`... |
| `la_capitule` |  |
| `la_repons` |  |
| `la_antienne_ev` |  |
| `la_antienne_ev2` | Antienne supplémentaire pour les dimanches du temps ordinaire (antienne spécifique du dimanche et à l’année liturgique). |
| `la_cantique_ev` |  |
| `la_antienne_evb` | Reprend la valeur de `l_antienne_ev`... |
| `la_antienne_ev2b` | Reprend la valeur de `l_antienne_ev2`... |
| `la_intercession` |  |
| `la_oraison` |  |
                            
### 3. Milieu du jour

| Id de l’élément `<div>` | Commentaire |
|---|---|
| `m_titre_principal` | Renseigné automatiquement par `lh.js` (date du jour) |
| `m_titre_sanctoral` | @ partir du fichier : `JJ-titre.html` |
| `m_titre_secondaire` | Renseigné automatiquement, contient les informations : temps liturgique, année liturgie, semaine, semaine du psautier. |
| `m_temporal` |  |
| `m_sanctoral` | Invisible, permet de charger le fichier `JJ-milieu.html` qui permet d’insérer le propre du saint. |
| `m_introduction` |  |
| `m_select-hymne-div` | Sélection de l’hymne en fonction de l’heure (tierce, sexte ou none) |
| `m_hymne` | Texte dont la valeur est fixée par la sélection ci-dessus |
| `m_antienne_1` |  |
| `m_psaume_1` |  |
| `m_antienne_1b` | Reprend la valeur de `m_antienne_1` / pas de variable de session |
| `m_antienne2` |  |
| `m_psaume2` |  |
| `m_antienne_2b` | Reprend la valeur de `m_antienne_2`... |
| `m_antienne3` |  |
| `m_psaume_3` |  |
| `m_antienne_3b` | Reprend la valeur de `m_antienne_3`... |
| `m_capitule1` |  |
| `m_repons1` |  |
| `m_oraison1` |  |
| `m_capitule2` |  |
| `m_repons2` |  |
| `m_oraison2` |  |
| `m_capitule3` |  |
| `m_repons3` |  |
| `m_oraison3` |  |
                        
### 4. Vêpres
| Id de l’élément `<div>` | Commentaire |
|---|---|
| `v_titre_principal` | Renseigné automatiquement par `lh.js` (date du jour) |
| `v_titre_sanctoral` | @ partir du fichier : `JJ-titre.html` |
| `v_titre_secondaire` | Renseigné automatiquement, contient les informations : temps liturgique, année liturgie, semaine, semaine du psautier. |
| `v_temporal` | Pour le temps particulier comme les féries du 17 au 24 décembre, l’octave de Noël ... permet le chargement d’un fichier qui personnalise les offices pour ces moments particuliers. |
| `v_sanctoral` | Invisible, permet de charger le fichier `JJ-laudes.html` qui permet d’insérer le propre du saint. |
| `v_introduction` | Automatique : *Dieu vient à mon aide*... |
| `v_antienne_inv` |  |
| `v_select_inv` | Permet de sélectionner le psaume invitatoire |
| `v_select-hymne-div` | Permet de sélectionner les offices quand plusieurs sont proposés. |
| `v_hymne` | Cette variable contient : soit l’url d’un fichier `html` qui contient le texte de l’hymne, soit le nom du fichier qui contient la liste des hymnes à choisir (liste qui permet de renseigner l’élément `select` de `la_select-hymne-div`) |
| `v_antienne_1` |  |
| `v_psaume_1` |  |
| `v_antienne_1b` | Reprend la valeur de `la_antienne_1` / pas de variable de session |
| `v_antienne_2` |  |
| `v_psaume_2` |  |
| `v_antienne_2b` | Reprend la valeur de `la_antienne_2`... |
| `v_antienne_cantique` |  |
| `v_cantique` |  |
| `v_antienne_cantiqueb` | Reprend la valeur de `la_antienne_cantique`... |
| `v_capitule` |  |
| `v_repons` |  |
| `v_antienne_ev` |  |
| `v_antienne_ev2` | Antienne supplémentaire pour les dimanches du temps ordinaire (antienne spécifique du dimanche et à l’année liturgique). |
| `v_cantique_ev` |  |
| `v_antienne_evb` | Reprend la valeur de `l_antienne_ev`... |
| `v_antienne_ev2b` | Reprend la valeur de `l_antienne_ev2`... |
| `v_intercession` |  |
| `v_oraison` |  |

## B. Participer au développement

### 1. Mise en place simple

Cette application s'appuie sur le framework **Cordova** d'Apache. Si on ouvre le fichier `index.html` dans son navigateur, rien ne se passe car le script `cordova.js` est manquant. Il y a plusieurs façons de régler ce problème. La plus simple est d'obtenir une copie de ce fichier. Si on souhaite éviter d'installer le package Nodejs de Cordova, une copie est disponible [ici](https://gist.github.com/a2ohm/e3ed0691b089ded4d6b53e22849790b5).

Enfin, pour respecter les politiques de sécurité des navigateurs, il faut aussi mettre en place un serveur local pour tester l'application sur son ordinateur ([source](https://developer.mozilla.org/fr/docs/Web/HTTP/CORS/Errors/CORSRequestNotHttp#chargement_dun_fichier_local)). Avec python, la commande `python3 -m http.server` exécutée dans le dossier local du projet suffit. L'application est alors accessible à l'adresse suivante `http://localhost:8000/` ([source](https://developer.mozilla.org/fr/docs/Learn/Common_questions/set_up_a_local_testing_server) ).

### 2. Mise en place complète de l'environnement de développement

#### a) Cordova
Installer Cordova.
- `sudo snap install node --classic`
- `npm install -g cordova`

**Rmq.** Une version de nodejs supérieure à la version 14 est nécessaire. Sur Ubuntu, la version du snapstore fait l'affaire, voir [nodejs.org](https://nodejs.org/en/download/package-manager) pour trouver ce qui est le plus adapté à votre distribution.

Créer le projet.
- `cordova create liturgie-des-heures`

Cloner le code source de l'application
- `cd liturgie-des-heures/www`
- `git clone https://github.com/lologhi/liturgie.git`

Mettre en place les plateformes sur lesquelles l'application sera testée.
- `cd ..`
- (navigateur) : `cordova platform add browser`
- (téléphone android) : `cordova platform add android`

#### b) Tester dans son navigateur
- `cordova build browser`
- `cordova run browser`

#### c) Tester sur son téléphone Android

**Pré-requis** ([source](https://stackoverflow.com/questions/34556884/how-to-install-android-sdk-on-ubuntu))

Installer le Java Development Kit.
- `sudo apt install default-jdk`

Télécharger les outils du sdk android depuis [ce site](https://developer.android.com/studio#command-tools) (chercher la section "Command line tools only").
Puis …
- `unzip commandlinetools-linux-8512546_latest.zip`
- `rm commandlinetools-linux-8512546_latest.zip`
- `mkdir -p android-sdk/cmdline-tools/`
- `mv cmdline-tools/ android-sdk/cmdline-tools/latest`

Ajouter dans `~/.bash_profile` :

    # Export the Android SDK path 
    export ANDROID_SDK_ROOT=$HOME/android-sdk
    export PATH=$PATH:$ANDROID_SDK_ROOT/platform-tools/
    export PATH=$PATH:$ANDROID_SDK_ROOT/build-tools/
    export PATH=$PATH:$ANDROID_SDK_ROOT/cmdline-tools/latest/bin

Puis …
- `source ~/.bash_profile`
- `sdkmanager "platform-tools" "platforms;android-30" "build-tools;30.0.3"`

*Rmq.*
- `platforms;android-30` − Pour choisir le numéro de version il suffit de ce reporter à [ce tableau](https://cordova.apache.org/docs/en/11.x/guide/platforms/android/index.html#android-api-level-support) avec le numéro de version de cordova-android (`cordova platform ls`).
- `build-tools;30.0.3` − C'est le numéro de version que me demande cordova dans le message d'erreur renvoyé par `cordova build android` lorsque les build-tools n'étaient pas encore installés.

Activer sur son téléphone le mode développeur (cliquer n fois sur le numéro de version du kernel).
Dans les options de développements, activer le débogage USB.
Brancher le téléphone. (Si besoin, dans le menu des paramètres USB, choisir l'option "USB contrôlé par cet appareil".)

**Tester**

- `cordova build android`
- `cordova run android`

*Rmq.* Il peut être nécessaire de débrancher rebrancher le téléphone avant la commande `cordova run android`. Aussi, bien vérifier que le débogage USB est activé.
