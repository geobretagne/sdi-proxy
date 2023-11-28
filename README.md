
# sdi-proxy

`sdi-proxy` est un un proxy-cache facile à installer pour rediffuser des flux géographiques de "fonds de carte" (plans , photographies...). Il est basé sur le logiciel [MapProxy](https://mapproxy.github.io/mapproxy/index.html). Il est préconfiguré avec les données de la  [Géoplateforme](https://www.ign.fr/geoplateforme) et des infrastructures locales de données de la communauté [geOrchestra](https://www.georchestra.org).


# Pourquoi

Les visualiseurs cartographiques sont dépendants des flux de "tuilage" qui servent plans et orthophotographies. En cas de panne de ceux-ci, l'utilisateur ne peut plus se repérer sur la carte et le visualiseur est pratiquement inutilisable. Il faut alors reconfigurer tous les contextes de carte, ce qui prend beaucoup de temps et n'est pas à la portée de tous les utilisateurs.


# Objectifs

L'objectif de `sdi-proxy` est de proposer un proxy-cache pour les flux tuilés, qui :

1. Est opérationnel en 5 minutes, avec un minimum de prérequis
1. fournit dès le premier démarrage un large panel de fonds de carte
1. présente des URLs stables alors que celles de la source peuvent changer
1. redémarre vite, consomme peu de ressources
1. peut subir une perte complète de stockage - logiciel et cache - et être à nouveau opérationnel rapidement
1. réduit la charge sur les plateformes distantes, ou diminue l'impact de leur indisponibilité
1. permet de débuter facilement avec MapProxy pour aller ensuite vers des configurations plus compliquées

Basé sur [MapProxy](https://github.com/mapproxy/mapproxy) en mode [MultiMapProxy](https://mapproxy.github.io/mapproxy/deployment.html#multimapproxy), sdi-proxy est déjà configuré pour servir les cartes orthophotopgraphies de la GéoPlateforme,
et les cartes OpenStreetMap de [GéoBretagne](https://geobretagne.fr) et [DataGrandEst](https://osm.datagrandest.fr/).

# Prérequis

`sdi-proxy` est testé sur ubuntu/debian/wsl. Il nécessite :
* python3.8-venv
* gdal-bin
* un espace de stockage de 20 Go minimum. L'usage de systèmes de fichiers réseau est déconseillé.


# Démarrage rapide

    apt install python3-pil python3-yaml python3-pyproj libgeos-dev
    mkdir sdi-proxy

    git clone https://git.geobretagne.fr/geobretagne/sdi-proxy/sdi-proxy.git

    cd sdi-proxy
    python3 -m venv venv
    source ./venv/bin/activate
    pip install --upgrade pip
    pip install -r requirements.txt

    # facultatif
    rmdir cache_data
    ln -s /mon/gros/espace/de/stockage/ cache_data

    ./debug

Pointez votre navigateur vers http://127.0.0.1:5000 pour voir la page d'accueil avec la configuration par défaut.

(screenshot homepage)

# Utiliser

## Les profils par défaut et leurs contenus

`sdi-proxy` propose de répartir les données en "profils". Chaque profil dispose de ses propres services.

Quelques profils sont proposés par défaut, libre à vous d'en rajouter ou de les compléter :

* `osm` offre une sélection de fonds de plan fabriqués avec les données OpenStreetMap. Ils proviennent de plusieurs plateformes pour améliorer la résilience du service.
* `geopf` offre les fonds de plan IGN, et plus généralement les plans disponibles par la GéoPlateforme.
* `photo` offre une sélection d'imageries aériennes et (à venir) satellite


| rendu | profil | identifiant | description | fournisseur |
|---|--------|-----------|-------------|--------|
|![osm:grey](https://tile.geobretagne.fr/osm/tms/osm:grey/EPSG3857/16/64287/85957.png)|`osm`|`osm:grey` |niveau de gris pour une meilleure lecture des données |GéoBretagne|
|![osm:default](https://tile.geobretagne.fr/osm/tms/osm:default/EPSG3857/16/64287/85957.png)|`osm`|`osm:default`|peu coloré|GéoBretagne|
|![osm:google](https://tile.geobretagne.fr/osm/tms/osm:google/EPSG3857/16/64287/85957.png)|`osm`|`osm:google`|ressemble à google maps, sans maisons|GéoBretagne|
|![osm:bing](https://tile.geobretagne.fr/osm/tms/osm:bing/EPSG3857/16/64287/85957.png)|`osm`|`osm:bing`|ressemble à bing maps, avec maisons|GéoBretagne|
|![osm:faded](https://tile.geobretagne.fr/osm/tms/osm:faded/EPSG3857/16/64287/85957.png)|`osm` |`osm:faded`|couleurs pastel|DataGrandEst|
|![osm:naturaliste](https://tile.geobretagne.fr/osm/tms/osm:naturaliste/EPSG3857/16/64287/85957.png)|`osm`|`osm:naturaliste`|avec éléments intéressant ce métier|DataGrandEst|
|![HR.ORTHOIMAGERY.ORTHOPHOTO](https://tile.geobretagne.fr/photo/tms/HR.ORTHOIMAGERY.ORTHOPHOTO/EPSG3857/16/64287/85957.png)|`photo`|`HR.ORTHOIMAGERY.ORTHOPHOTO`|orthophotographie 20cm|Géoplateforme|
|![ORTHOIMAGERY.ORTHOPHOTOS.IRC](https://tile.geobretagne.fr/photo/tms/ORTHOIMAGERY.ORTHOPHOTOS.IRC/EPSG3857/16/64287/85957.png)|`photo`|`ORTHOIMAGERY.ORTHOPHOTOS.IRC`|orthophotographie proche infrarouge 20cm|Géoplateforme|
|![ORTHOIMAGERY.ORTHOPHOTOS.1950-1965](https://tile.geobretagne.fr/photo/tms/ORTHOIMAGERY.ORTHOPHOTOS.1950-1965/EPSG3857/16/64287/85957.png)|`photo`|`ORTHOIMAGERY.ORTHOPHOTOS.1950-1965`|première campagne photo aérienne intégrale de la France|Géoplateforme
|![GEOGRAPHICALGRIDSYSTEMS.PLANIGNV2](https://tile.geobretagne.fr/geopf/tms/GEOGRAPHICALGRIDSYSTEMS.PLANIGNV2/EPSG3857/16/64287/85957.png)|`geopf`|GEOGRAPHICALGRIDSYSTEMS.PLANIGNV2`|plan fabriqué automatiquement à partir des données IGN|Géoplateforme|




## Les adresses de services

On suppose ici que `sdi-proxy` publie sur https://www.maplateforme.net/, et que le le profil est `osm`. Les points d'entrée des services sont :

### WMS 1.3.0

Le `Web Map Service` fournit des cartes pour n'importe quelle emprise. Il est supporté par presque tous les logiciels cartographiques. Cependant est beaucoup plus lent que les services tuilés et ne devrait pas être utilisé pour servir plans et photographies.

* capacités WMS : https://www.maplateforme.net/`osm`/service?SERVICE=WMS&REQUEST=GetCapabilities

* point d'entrée du service : https://www.maplateforme.net/`osm`/service?

### WMTS 1.0.0

Le `Web Map Tiled Service` fournit des cartes selon des emprises décrites dans une grille, ce qui permet de les générer à l'avance. Il est beaucoup plus rapide que `WMS` mais demande au client de connaitre la grille de tuilage.

* capacités WMS : https://www.maplateforme.net/`osm`/service?SERVICE=WMTS&REQUEST=GetCapabilities

* point d'entrée du service : https://www.maplateforme.net/`osm`/service?

### TMS 1.0.0

Le `Tiled Map Service` est similaire au WMTS avec un formalisme plus simple.

* point d'entrée du service : https://www.maplateforme.net/`osm`/tms/1.0.0


# Configuration de quelques clients

## Miewer

[Mviewer](https://mviewer.netlify.app/fr/) est un visualiseur cartographique léger et entièrement personnalisable.

## Mapfishapp

## Mapstore

## OpenLayers

## Leaflet

## QGIS

# Production

## Reverse proxy (Nginx)

Inclure le fragment suivant dans la configuration de l'hôte :

    location / {
       proxy_pass http://hostnamehere:5000;
       proxy_set_header   Host             $host;
       proxy_set_header   X-Real-IP        $remote_addr;
       proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
    }

Avec un chemin, par exemple : `/tuileur` :

    location /tuileur {
       proxy_pass http://hostnamehere:5000;
       proxy_set_header   Host             $host;
       proxy_set_header   X-Script-Name    /tuileur;
       proxy_set_header   X-Real-IP        $remote_addr;
       proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
    }

## Reverse proxy (autres)

Se référer à la documentation https://mapproxy.github.io/mapproxy/deployment.html

## Service systemd

    [Unit]
    Description=sdi-proxy via gunicorn
    After=network.target
    
    [Service]
    PIDFile=/tmp/sdi-proxy.pid
    User=www-data
    Group=www-data
    WorkingDirectory=/path/to/sdi-proxy/
    ExecStart=/path/to/sdi-proxy/venv/bin/gunicorn -w 8 -b 0.0.0.0:5000 --pid=/tmp/sdi-proxy.pid config:application
    ExecReload=/bin/kill -s HUP $MAINPID
    ExecStop=/bin/kill -s TERM $MAINPID
    Restart=always
    
    [Install]
    WantedBy=multi-user.target

# Crédits

* [MapProxy](https://mapproxy.github.io/) et ses nombreux contributeurs
* [Géoplateforme]() : orthophotographies, plan
* [DataGrandEst](https://www.datagrandest.fr/) : carte OpenStreetMap
* [GéoBretagne](https://geobretagne.fr/) : carte OpenStreetMap, tests
* [Georchestra](https://www.georchestra.org/) et sa communauté : tests et soutien
* [Mviewer]() pour les tests

# Licence

# Comment contribuer

Les contributions sont les bienvenues. Les propositions doivent respecter les considérations suivantes :

* la reconfiguration des clients est coûteuse et doit être réduite au maximum
* les données doivent couvrir au moins la France métropolitaine
* les caches peuvent être purgés à tout moment, sans préjudice autre qu'une réduction de performances



