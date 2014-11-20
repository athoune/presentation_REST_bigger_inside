!SLIDE
#Le monde est REST

La guerre des services est finie, REST a gagné.

Les appels distants utilisent HTTP, et c'est comme ça.

REST commence déjà à utiliser Websocket, et il utilisera HTTP2 dès qu'il le pourra.

Curl est son herault.

!SLIDE
#HTTP guérit tout

* Il est normalisé
* Il est stateless
* Les firewalls l'aiment bien
* Il se proxyfie très bien
* Les headers sont bien pratiques
* Il gère très bien les gros volumes et la reprise en cas de vautrage
* Il peut gérer du stream
* L'authentification peut être ajoutée après
* Il y a plein de devs pouvant le martyriser
* Il passe bien sur Internet

HTTP est battery include

!SLIDE
#As A Service

Pour des services proposant des latences se mesurant en dizaine de millisecondes, HTTP est plus que suffisant.

Il n'y a pas de problème pour trouver des drivers en HTTP
(par contre, pour en trouver des bons, c'est un poil plus compliqué, comme le
prouve tous les jours Ruby).

!SLIDE
#À la maison

Les webservices ne se trouvent pas que sur Internet, ils ont tout autant leur place à la maison.

 * CouchDB
 * Riak
 * Swift (le S3 d'OpenStack)
 * Elasticsearch
 * …
 * Pas mal de code métier, en fait, consommé par du HTML5

!SLIDE
#Débuguer le bazar

curl est fort pratique pour bidouiller ou comprendre la doc.

Mais le code va utiliser un drivers, qui emballe le tout et expose quelque
chose respectant l'ambiance du langage.

Le drivers abstrait le protocole et sa couche de communication.
L'application va générer des tonnes de requêtes, le serveur va se comportement
différemment en charge et les soucis commencent à apparaitre.

Les serveurs savent exposer leur état, pour trouver là où ils ont mal,
mais ils ont du mal à parler de la qualité de leurs communications.

Les archis distribués n'arrangent pas la visibilité de l'ensemble.

!SLIDE
#Lire le flot de log

`tail -f` est notre ami, mais les logs, pour peu qu'ils existent sont souvent
conçu pour webalizer.
Aucune notion de temps de réponse (du worker), des header/body des requêtes et
des réponses, d'un identifiant de connexion.

!SLIDE
#It's bigger inside

![Tardis](tardis.jpg)

!SLIDE
#Packetbeat

Packetbeat a un nom difficile à prononcer sans ricaner par un français.

Packetbeat est une sonde en Go utilisant pcap (tcpdump, Wireshark)
pour espionner les échanges passant sur la carte réseau.

Il sait lire pas mal de protocoles, et il transfère ses trouvailles en JSON
sur une machine qui l'analysera.

La cible officielle est Elasticsearch, mais Redis (comme queue ou comme pubsub)
est aussi utilisable.

!SLIDE
#Ajouter un peu de finesse

Le travail de packetbeat coté client est volontairement réduit, pour limiter
la gène sur son hôte.

Les échanges sont envoyés sur un pubsub.

Différents analyseurs sont branchables sur le pubsub, pour afficher, loguer,
grapher, voir même déclencher des actions.

Il faut lire le bloc de headers, parser le body qui peut avoir le bon gout d'être en JSON.

Rien d'infaisable avec les logithèques actuels.

!SLIDE
#Surveiller et punir Elasticsearch

Elasticsearch utilise massivement du REST pour interagir avec l'extérieur
(il se réserve un protocole binaire pour discuter entre ses noeuds).

Certains détails, comme les retours d'erreurs ou les tailles de bulks sont plus ou moins masqués au développeur.
Plus que moins, quand on utilise Logstash.

!SLIDE
#La politesse du client

La plupart des clients se présentent, avec un user agent. Pas logstash.

Les plugins (kopf, head…) sont proxifiés, et ils ont le user agents du butineur.

La paire IP/port du client permet de repérer ceux qui gardent les connexions ouvertes (ou partage un pool de connexion).

!SLIDE
#Gare de triage

Elasticsearch a des usages divers, comme la classique recherche plein-texte, mais aussi comme mangeur de log.

Les différents usages ont des temps de réponse complètement différents.

Il faut redécouper les urls pour retrouver les actions et les indices concernés.

!SLIDE
#Afficher des courbes

Statsd est l'ami des gens qui comptent. Graphite assure l'affichage.

![Graphite](graphite_elasticsearch.png)

!SLIDE
#Trouver des coupables

Les courbes sont pratiques pour trouver des patterns, et pour égayer une présentation.

On peut les empiler avec d'autres types de mesures pour trouver des corrélations,
avec les consommations de ressources physiques, par exemple.

Mais on peut aussi ne pas voir grand-chose, ou des choses trop vagues.

Les logs sont alors bien pratiques pour trouver des coupables.

À défaut de coupable, on peut loguer plus de choses, ou mouliner les résultats, pour les faire parler.

!SLIDE
#Et Kibana + Logstash?

Surveiller du Elasticsearch avec un autre Elasticsearch, j'ai eu peur du larsen, en fait.
Niveau encombrement, Graphite/Carbon/Statsite est bien plus discret qu'un Elasticsearch.

Logstash propose, un joli lot de modules utilisables dés la sortie du carton,
et sa gestion des threads le rends difficilement concurrençable.

Je voulais rajouter du code métier, avec des itérations courtes, sans enrichir le contenu.

Un peu la flemme de faire un module jruby, en fait.

1% de CPU, 25Mo de RAM, Logstash a du mal à rivaliser avec du python (sauvé par gevent).



!SLIDE
#Elasticstat

Le code utilisé pour dessiner ce si joli graphique est sur github (et en python).

https://github.com/bearstech/elasticstat


