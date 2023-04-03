# 🖥 GNU Linux Avancé

*👨‍💻 Charles, Justin, Guillian*

**Grafana 👉 https://m7.vms.re:3000/**

**Elasticsearch 👉 https://m7.vms.re:5601/ (bypass token : http://m7.vms.re:5601/?code=599355)**


# Activité 2


***1. Que signifie p95, p99 ? Pourquoi ne pas utiliser les moyennes de temps ?***

- p95 et p99 représente le RPS percentile latency
- si l'on souhaite un p95 à 30ms, cela signifie que sur l'ensemble de nos requetes, 95% de celles-ci sont en dessous de 30ms
- Meme chose pour p99, 99% de nos requettes doivent être en dessous de 30ms

- On utilise pas les moyennes de temps car...


***2. Quelles sont les autres contraintes que celles de temps sur le système***

- La performance


***3. Plan de test***

On souhaite que p95 = 300ms et p99 = 800ms

| Route | Nombre de connexion simultanées | Nombre de requêtes | p95 | p99 | Observation                                    |
| ----- |:------------------------------- |:------------------ |:--- |:--- |:---------------------------------------------- |
| p1    | 1000                            | 20000              | 209 | 234 | On peut encore pousser mais c'est déjà pas mal |
| p2    | 10                              | 200                | 303 | 503 |                                                |
| p3    | 50                              | 1500               | 308 | 397 |                                                |
| p4    | 3                               | 4000               | 221 | 258 | Mange beaucoup de CPU et de RAM                |


# Activité 3


***1. Qu’est ce que Loki ?***

Loki est un ensemble de composants qui peuvent être composés en une pile de logs complète.

Contrairement à d'autres systèmes de logs, Loki est construit autour de l'idée d'indexer uniquement les métadonnées de vos journaux : les labels (comme les labels Prometheus). Les données des logs sont ensuite compressées et stockées par chunk dans des store d'objets tels que S3 ou GCS, ou même localement sur le système de fichiers. Un petit index et des chunk hautement compressés simplifient l'opération et réduisent considérablement le coût de Loki.


***2. Quelles sont les différentes composantes de la stack Loki fournie ?***

- loki en mode write
- loki en mode read
- promtail
- minio
- nginx
- flog


***3. A quoi servent chacunes des composantes ?***

- Loki : Système d'aggrégation de logs visant à permettre de stocker et rechercher parmi les logs de toutes les applications et infrstructures que l'on détient.  
- Promtail : Permet d'envoyer le contenu de logs sauvegardés en local vers une instance privée Grafana Loki et/ou Cloud.
- Minio : Minio permet le stockage d'objets (même sur S3).
- NGINX : Nginx est un logiciel de serveur web faisant également office de reverse proxy, load balancer, proxy mail et cache HTTP.
- FLOG : Permet de générer de faux logs, particulièrement utile pour tester un système de logging.


***4. Qu'est ce que Minio***

Minio est un système de stockage d'objet haute performance. Il est possible de définir un volume de sockage et de le faire apparaitre en tant que stockage S3.


***5. Réalisez un schéma (à mettre dans le dossier partie1 du repo) du flux de données entre le moment où un log est écrit dans docker jusqu’au moment où il est visible dans grafana.***

écriture des logs

--> flog génère un log
--> promtail POST --> /loki/api/v1/push
--> nginx redirect sur loki write
--> loki config avec minio qui écrit les logs dans "loki-ruler" (format S3)

lecture des logs

--> grafana GET --> /loki/api/$request (/loki/api/v1/query_range?...)
--> nginx redirect sur loki read


***6. À quoi sert le service gateway ?***

Le service gateway est le serveur nginx qui va servir de reverse proxy pour permettre la lecture ou l'écriture de logs.

https://grafana.com/docs/grafana/latest/explore/logs-integration/


***7. L’application a une structure de logs plutôt constante, quels champs sont contenus dans les logs ? Quelle données représentent-il?***

- Time : Timestamp de l'évènement
- evt : De quel type d'évènement il s'agit
- level : Niveau de criticité de l'erreur
- msg : Message affilié au log
- name : Nom du produit concerné par le log
- product : id du produit
- stocks : Combien de produits sont encore en stocks
- time : Temps en chaîne de charactère sous format Y-m-dTH:i:s
- tsNs : Timestamp en nanosecondes
- uuid : ID de l'utilisateur


***8. Qu’est-ce qui doit être configuré du coté de la stack loki pour récolter les logs ?***

On doit le pluger à tous les containers de logs. Promtail se chargera alors de récupérer les logs.


***9. Dans l’onglet “explorer”, quelle requête permet d’avoir :***
  ***1. le volume de logs par couleur***
  ***2. Filtrer les resultats pour un utilisateur***

1. **`sum by(level) (count_over_time({container="logapp"} | json [1h]))`**
2. **```{container="logapp"} |= `` | json | uuid = `0b08d5ef-bda3-4e62-af79-ce8e8a99e0c2` ```** (changer l'uuid selon l'utilisateur)


***10. Créer un dashboard qui contiendra la part de paiements en succès et en échec***

![](https://i.imgur.com/sieOMvr.png)


***11. qu’est-ce qu’elasticsearch ?***

Elasticsearch est un moteur de recherche et d'analyse de données distribué, basé sur Lucene, qui permet de stocker, d'interroger et d'analyser des données structurées et non structurées en temps réel.


***12. À quoi sert elasticsearch ?*** 

Elasticsearch est utilisé pour la recherche et l'analyse de données à grande échelle. Il permet de rechercher rapidement des données dans de grands ensembles de données, de les analyser et de les visualiser.


***13. Qu’est ce que Kibana ? Sur quel port accéder à Kibana ?***

Kibana est une plateforme de visualisation et d'analyse de données open-source qui fonctionne avec Elasticsearch. Kibana permet de visualiser et d'analyser les données stockées dans Elasticsearch en créant des tableaux de bord et des visualisations interactives.


***14. Qu’est-ce qu’un index Elasticsearch ?***

Dans Elasticsearch, un index est un ensemble de documents qui ont des caractéristiques similaires. Il peut être comparé à une table dans une base de données relationnelle. Chaque document dans l'index est une unité de données individuelle qui peut être indexée et recherchée.


***15. Qu’est-ce que Lucene ? Quel est son rapport avec Elasticsearch ?***

Lucene est une bibliothèque de recherche d'informations open-source, également créée par Apache. Elasticsearch utilise Lucene pour effectuer des recherches, des analyses de texte, de la recherche de phrases et de la recherche de proximité. Elasticsearch s'appuie sur Lucene pour la plupart de ses fonctionnalités de recherche et d'indexation.


***16. Qu’est-ce que Filebeat ?***

Filebeat permet de collecter des données et de rediriger le flux vers Elasticsearch ou Logstash pour l'indexation et l'analyse.


***17. Quels autres “beats” existent ?***

Metricbeat : pour la collecte de données de métriques système et d'application.
Packetbeat : pour la collecte de données de réseau.
Winlogbeat : pour la collecte de données d'événements Windows.
Auditbeat : pour la collecte de données d'audit système et de sécurité.
Heartbeat : pour la surveillance des services et des serveurs.
Functionbeat : pour la collecte de données à partir de fonctions cloud, telles que AWS Lambda.
