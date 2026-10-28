# Atelier 5 — API REST et documentation OpenAPI

## A. Endpoints

Les contrôleurs délèguent exclusivement à leur interface `I...Service`. Le
préfixe `/api` est ajouté une seule fois par `server.servlet.context-path`.

| Contrôleur | Verbe et route | Service appelé | Succès | Erreurs |
|---|---|---|---|---|
| Agence | `GET /api/agences` | `IAgenceService.findAll` | 200 | — |
| Agence | `GET /api/agences/{id}` | `IAgenceService.findById` | 200 | 404 |
| Agence | `POST /api/agences` | `IAgenceService.create` | 201 | 400 |
| Agence | `PUT /api/agences/{id}` | `IAgenceService.update` | 200 | 400, 404 |
| Agence | `DELETE /api/agences/{id}` | `IAgenceService.deleteById` | 204 | 404 |
| Employé | `GET /api/employes` | `IEmployeService.findAll` | 200 | — |
| Employé | `GET /api/employes/{id}` | `IEmployeService.findById` | 200 | 404 |
| Employé | `POST /api/employes` | `IEmployeService.create` | 201 | 400 |
| Employé | `PUT /api/employes/{id}` | `IEmployeService.update` | 200 | 400, 404 |
| Employé | `DELETE /api/employes/{id}` | `IEmployeService.deleteById` | 204 | 404 |
| Véhicule | `GET /api/vehicules` | `IVehiculeService.findAll` | 200 | — |
| Véhicule | `GET /api/vehicules/{id}` | `IVehiculeService.findById` | 200 | 404 |
| Véhicule | `POST /api/vehicules` | `IVehiculeService.create` | 201 | 400 |
| Véhicule | `PUT /api/vehicules/{id}` | `IVehiculeService.update` | 200 | 400, 404 |
| Véhicule | `DELETE /api/vehicules/{id}` | `IVehiculeService.deleteById` | 204 | 404 |
| Équipement | `GET /api/equipements` | `IEquipementService.findAll` | 200 | — |
| Équipement | `GET /api/equipements/{id}` | `IEquipementService.findById` | 200 | 404 |
| Équipement | `POST /api/equipements` | `IEquipementService.create` | 201 | 400 |
| Équipement | `PUT /api/equipements/{id}` | `IEquipementService.update` | 200 | 400, 404 |
| Équipement | `DELETE /api/equipements/{id}` | `IEquipementService.deleteById` | 204 | 404 |
| Client | `GET /api/clients` | `IClientService.findAll` | 200 | — |
| Client | `GET /api/clients/{id}` | `IClientService.findById` | 200 | 404 |
| Client | `POST /api/clients` | `IClientService.create` | 201 | 400 |
| Client | `PUT /api/clients/{id}` | `IClientService.update` | 200 | 400, 404 |
| Client | `DELETE /api/clients/{id}` | `IClientService.deleteById` | 204 | 404 |
| Réservation | `GET /api/reservations` | `IReservationService.findAll` | 200 | — |
| Réservation | `GET /api/reservations/{id}` | `IReservationService.findById` | 200 | 404 |
| Réservation | `POST /api/reservations` | `IReservationService.create` | 201 | 400 |
| Réservation | `PUT /api/reservations/{id}` | `IReservationService.update` | 200 | 400, 404 |
| Réservation | `DELETE /api/reservations/{id}` | `IReservationService.deleteById` | 204 | 404 |
| Contrat | `GET /api/contrats` | `IContratService.findAll` | 200 | — |
| Contrat | `GET /api/contrats/{id}` | `IContratService.findById` | 200 | 404 |
| Contrat | `POST /api/contrats` | `IContratService.create` | 201 | 400 |
| Contrat | `PUT /api/contrats/{id}` | `IContratService.update` | 200 | 400, 404 |
| Contrat | `DELETE /api/contrats/{id}` | `IContratService.deleteById` | 204 | 404 |
| Paiement | `GET /api/paiements` | `IPaiementService.findAll` | 200 | — |
| Paiement | `GET /api/paiements/{id}` | `IPaiementService.findById` | 200 | 404 |
| Maintenance | `GET /api/maintenances` | `IMaintenanceService.findAll` | 200 | — |
| Maintenance | `GET /api/maintenances/{id}` | `IMaintenanceService.findById` | 200 | 404 |
| Maintenance | `POST /api/maintenances` | `IMaintenanceService.create` | 201 | 400 |
| Maintenance | `PUT /api/maintenances/{id}` | `IMaintenanceService.update` | 200 | 400, 404 |
| Maintenance | `DELETE /api/maintenances/{id}` | `IMaintenanceService.deleteById` | 204 | 404 |

`PaiementController` ne propose que la lecture : les créations et suppressions
passent par le contrat et sa cascade. Son absence de route POST entraîne une
réponse 405. Les erreurs métier sont converties en JSON `{statut, message,
horodatage}` par `GlobalExceptionHandler`.

## B. Sérialisation Jackson

| Association | Annotation | Justification |
|---|---|---|
| `Contrat.paiements` / `Paiement.contrat` | `@JsonManagedReference` / `@JsonBackReference` avec le même nom | Le contrat est le parent de composition ; Jackson sérialise ses paiements sans reboucler vers lui et restaure le parent lors de la lecture du JSON. |
| Collections inverses d'agence, client, véhicule et équipement | `@JsonIgnore` | Ces collections ne sont pas nécessaires pour créer leurs ressources ; elles évitent les cycles en conservant les associations propriétaires lisibles dans les requêtes. |
| Associations vers véhicule | `@JsonIgnoreProperties` sur certaines références | Empêche de ré-énumérer ses collections lors d'une réponse imbriquée. |

`@JsonIgnore` omet complètement une propriété à la lecture comme à l'écriture.
`@JsonBackReference` marque le côté retour d'une paire : il est omis lors de la
sérialisation et rattaché au parent lors de la désérialisation. La collection
`Agence.vehicules` est ignorée car l'agence n'est pas le parent de composition
des véhicules ; à l'inverse, le contrat sérialise ses paiements, créés en même
temps que lui par cascade. Le modèle JPA ne comporte pas de quantité sur
`Equipement`, ni de coût/kilométrage sur les entités concernées : seules les
règles réalisables avec les champs présents sont appliquées.

## C. Postman et Swagger UI

Postman envoie et rejoue des requêtes concrètes : par exemple, créer un contrat
avec ses paiements, vérifier le 201, puis constater les paiements dans
`GET /api/paiements`. Swagger UI présente le contrat OpenAPI généré depuis les
contrôleurs et permet d'essayer les routes sans quitter la documentation.
Adresses : `/api/v3/api-docs` et `/api/swagger-ui.html`.

## D. Anomalies SonarQube for IDE

SonarQube for IDE n'est pas disponible dans cet environnement ; aucun résultat
d'analyse réel n'est revendiqué. À exécuter dans IntelliJ sur
`web.controller` et à compléter après inspection du panneau :

| Règle constatée | Explication | Correction |
|---|---|---|
| À relever dans SonarQube for IDE | Noter la règle et le contrôleur concerné. | Noter le changement réellement effectué. |
| À relever dans SonarQube for IDE | Noter une seconde anomalie réelle. | Noter sa correction. |

## E. Questions de compréhension

1. Le `DispatcherServlet` reçoit les requêtes HTTP et les associe à une méthode
   en fonction du chemin et du verbe déclarés par les mappings Spring MVC.
2. `ContratController` dépend de `IContratService` pour rester indépendant de
   l'implémentation et du repository ; la logique métier demeure dans le service.
3. POST renvoie 201 car une ressource est créée ; DELETE renvoie 204 car
   l'opération a réussi sans contenu de réponse. Une ressource absente produit
   404 via `ResourceNotFoundException`, et non une erreur serveur 500.
4. Les paiements appartiennent au contrat et sont créés/supprimés par sa
   cascade : le contrôleur de paiement est en lecture seule. Un POST non mappé
   sur `/api/paiements` renvoie 405.
5. Exposer les entités lie l'API au schéma JPA, peut divulguer des champs et
   complique les associations ; les DTO sépareront ces contrats au chapitre 10.
   `Contrat.paiements` utilise une référence gérée pour exposer la composition,
   tandis que `Agence.vehicules` est ignorée pour éviter un graphe cyclique qui
   n'est pas requis pour créer un véhicule avec son agence.
