# Atelier 3 — Repositories Spring Data JPA

## Choix des interfaces

Les neuf interfaces étendent `JpaRepository<Entite, Long>`. Cette interface
fournit le CRUD complet, des résultats `List` pour les recherches multiples,
le tri et la pagination, ainsi que `flush` et `saveAndFlush`. `Long` est le
type des identifiants générés des entités AutoLoc.

| Entité | Repository |
|---|---|
| Agence | `IAgenceRepository` |
| Employe | `IEmployeRepository` |
| Vehicule | `IVehiculeRepository` |
| Equipement | `IEquipementRepository` |
| Client | `IClientRepository` |
| Reservation | `IReservationRepository` |
| Contrat | `IContratRepository` |
| Paiement | `IPaiementRepository` |
| Maintenance | `IMaintenanceRepository` |

`CrudRepository` convient au CRUD de base, mais `findAll()` renvoie un
`Iterable`. `ListCrudRepository` conserve le CRUD avec des résultats `List`.
`PagingAndSortingRepository` fournit le tri et la pagination, sans hériter du
CRUD depuis Spring Data 3. `JpaRepository` est retenue ici car elle réunit le
CRUD, les listes, le tri, la pagination et les opérations JPA.

## Utilisation correcte du CRUD

- `save(entite)` persiste une nouvelle entité lorsque son identifiant est nul
  et renvoie l'entité enregistrée (avec son identifiant généré). Pour une
  entité existante, la mise à jour peut passer par `merge` : utiliser l'objet
  renvoyé par `save`, qui peut être une instance distincte de celle fournie.
- `findById(id)` renvoie un `Optional<Entite>`. Traiter explicitement
  l'absence, par exemple avec `orElseThrow(...)`, plutôt que d'appeler
  `get()` sans vérifier.
- `deleteById(id)` ne signale pas d'erreur si l'identifiant est absent dans
  Spring Data 3. La suppression standard s'effectue dans le contexte de
  persistance et applique les cascades JPA configurées.
- `saveAndFlush(entite)` enregistre et synchronise immédiatement les
  modifications en attente avec la base. Il ne remplace pas une transaction.
- Éviter `deleteAllInBatch` et `deleteAllByIdInBatch` pour les contrats :
  ces suppressions en lot contournent le contexte de persistance et les
  cascades/orphan removal.

Pour observer le comportement SQL, configurer `spring.jpa.show-sql=true` et
`logging.level.org.hibernate.SQL=DEBUG`, démarrer l'application avec une
connexion MySQL valide, puis consulter les logs lors des appels repository.
Un INSERT correspond à la persistance d'une nouvelle entité ; un SELECT peut
précéder la mise à jour d'une entité existante ; DELETE indique une
suppression. La visibilité des écritures dépend notamment du flush et de la
transaction.

Pour Contrat–Paiement, les paiements doivent être ajoutés ou retirés via leur
contrat afin de respecter la composition et la cascade définies dans le
modèle. `IPaiementRepository` peut notamment être utilisé pour les lectures.

## Qualité et conventions

Les entités utilisent des annotations Lombok ciblées (`@Getter`, `@Setter`,
`@NoArgsConstructor`, `@AllArgsConstructor`) plutôt que `@Data`, afin d'éviter
notamment les méthodes `toString`, `equals` et `hashCode` générées sur les
relations JPA.

SonarQube for IDE n'a pas été exécuté dans le cadre de ce travail. Il serait
incorrect d'affirmer que des anomalies ont été détectées ou corrigées sans
résultats d'analyse réels. Après analyse dans IntelliJ IDEA, compléter ce
relevé avec les problèmes effectivement signalés et les corrections réalisées :

| Anomalie observée | Règle / explication SonarQube for IDE | Correction apportée |
|---|---|---|
| À compléter après l'analyse réelle | À compléter | À compléter |
| À compléter après l'analyse réelle | À compléter | À compléter |
| À compléter après l'analyse réelle | À compléter | À compléter |
