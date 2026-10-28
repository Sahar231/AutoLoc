# Atelier 4 — IoC, injection de dépendances et couche Service

## A. Modes d'injection comparés

| Critère | Constructeur (`@RequiredArgsConstructor`) | Attribut (`@Autowired`) |
|---|---|---|
| Champ `final` | Oui | Non |
| Dépendance visible dans la signature | Oui, dans le constructeur généré | Non |
| Utilisation hors conteneur | Possible en fournissant la dépendance | Le champ reste `null` |
| SonarQube for IDE | Bonne pratique retenue | L'exercice demande de vérifier S6813 |

L'injection par constructeur rend les dépendances obligatoires et permet de garder
les services sans état. Tous les services AutoLoc utilisent un champ `final`
typé par l'interface du repository et le constructeur généré par Lombok.

## B. Services

| Service | Dépendance injectée | Mode | Justification |
|---|---|---|---|
| `AgenceServiceImpl` | `IAgenceRepository` | Constructeur | CRUD et nom obligatoire |
| `EmployeServiceImpl` | `IEmployeRepository` | Constructeur | CRUD et nom obligatoire |
| `VehiculeServiceImpl` | `IVehiculeRepository` | Constructeur | CRUD et tarif non négatif |
| `EquipementServiceImpl` | `IEquipementRepository` | Constructeur | CRUD et libellé obligatoire |
| `ClientServiceImpl` | `IClientRepository` | Constructeur | CRUD et nom/e-mail obligatoires |
| `ReservationServiceImpl` | `IReservationRepository` | Constructeur | CRUD et dates cohérentes |
| `ContratServiceImpl` | `IContratRepository` | Constructeur | CRUD, montant non négatif |
| `PaiementServiceImpl` | `IPaiementRepository` | Constructeur | Lecture seule ; écritures via le contrat |
| `MaintenanceServiceImpl` | `IMaintenanceRepository` | Constructeur | CRUD et dates cohérentes |

Pour chaque CRUD, `create` refuse un identifiant déjà renseigné, `findById` et
`deleteById` signalent une ressource absente avec `ResourceNotFoundException`,
et `update` charge l'entité existante avant de recopier les champs scalaires
modifiables ; les associations et collections sont conservées. Les écritures
sur les relations sont laissées au chapitre dédié à leur gestion.
Le modèle `Equipement` ne contient pas de quantité ; seule la règle applicable
au libellé peut donc être vérifiée sans modifier le modèle de l'atelier précédent.
Les véhicules ne comportent pas de kilométrage ; le tarif journalier est le
champ numérique contrôlé.

## C. Messages d'erreur du conteneur

1. **`NoSuchBeanDefinitionException` pour `IContratService`** : `ContratServiceImpl`
   n'est probablement pas annotée `@Service`, ou elle est placée hors du package
   `tn.esprit.autoloc` et de ses sous-packages. Ajouter l'annotation et vérifier
   le package de scan.
2. **`NoUniqueBeanDefinitionException`** (`emailNotificateur`, `smsNotificateur`) :
   plusieurs beans satisfont le type. Placer `@Primary` sur la classe du bean par
   défaut, ou `@Qualifier("emailNotificateur")` sur le paramètre du constructeur
   du consommateur.
3. **`BeanCurrentlyInCreationException`**, dépendance circulaire entre services :
   les deux constructeurs exigent chacun l'autre service, donc Spring ne peut pas
   en construire un premier. Extraire la responsabilité commune dans un service
   tiers partagé, de sorte que les deux services en dépendent sans se dépendre
   mutuellement.

## D. Anomalies SonarQube for IDE

L'analyse SonarQube for IDE dépend du plugin IntelliJ et n'a pas été exécutée
dans cet environnement ; les constats ci-dessous ne prétendent pas être des
résultats d'analyse. L'exercice indique les deux règles à vérifier lors des
expériences guidées :

| Règle / constat | Explication | Correction |
|---|---|---|
| S6813 (à vérifier dans l'IDE) | L'injection par attribut est déconseillée car elle masque les dépendances et empêche un champ `final`. | Remplacée par le constructeur généré à partir du champ `final`. |
| S112 (à vérifier dans l'IDE) | Une exception générique telle que `RuntimeException` ne décrit pas l'erreur métier. | `findById` et `deleteById` utilisent `ResourceNotFoundException`. |
| À compléter après l'analyse réelle | Le troisième constat demandé doit provenir du panneau SonarQube for IDE, et non être supposé. | Consigner ici la règle et la correction réellement appliquée. |

## E. Questions de compréhension

1. `ContratServiceImpl` n'est pas construit avec un `new` écrit dans l'application :
   le conteneur Spring l'instancie au démarrage après détection de `@Service`, puis
   lui injecte le proxy associé à `IContratRepository`.
2. Un contrôleur dépend de `IContratService` pour rester couplé à un contrat
   abstrait et pouvoir remplacer l'implémentation sans modifier le contrôleur.
3. Un singleton est partagé entre les requêtes et les threads. Stocker un
   « contrat courant » dans un attribut modifiable pourrait faire utiliser à une
   requête les données d'une autre.
4. `update` charge l'entité persistée afin de ne modifier que les champs voulus.
   Passer directement l'objet reçu à `save` peut écraser des colonnes ou des
   relations omises, notamment la collection des paiements du contrat.
5. `@Component` déclare une classe détectable par scan ; `@Bean` déclare
   explicitement, dans une classe `@Configuration`, le résultat d'une méthode.
   `@Primary` choisit le candidat par défaut sur une classe, tandis que
   `@Qualifier` choisit explicitement un bean au point d'injection. Les paiements
   sont composés par le contrat et supprimés en cascade : `IPaiementService`
   n'expose donc que la lecture, les écritures passant par le contrat.
