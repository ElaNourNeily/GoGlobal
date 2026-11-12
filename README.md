# GoGlobal

*Plateforme de stages et de mobilité académique*  
Mini-projet AWD · Année universitaire 2026–2027 · Validation intermédiaire

GoGlobal aide les étudiants à trouver un stage, un échange universitaire, un double diplôme ou un master à l’étranger. Les offres et leurs conditions sont souvent dispersées. La plateforme les rassemble et aide chaque étudiant à répondre à deux questions : « Quelles offres correspondent à mon parcours ? » et « Quelles pièces dois-je encore fournir pour candidater ? »

## Acteurs et parcours

- *L’étudiant* renseigne ses expériences, ajoute ses justificatifs et consulte les offres compatibles avec son parcours. Il dépose ensuite un dossier et suit son avancement jusqu’à la décision finale.
- *L’entreprise ou l’université partenaire* publie une offre, précise ses critères et les pièces exigées, puis organise la sélection.
- *Le jury* évalue les dossiers selon un barème, valide la liste préliminaire, convoque les candidats et enregistre les résultats.
- *L’administrateur* gère les profils et les affiliations qui autorisent un utilisateur à agir pour un organisme.

Par exemple, Lina découvre un échange en informatique grâce à un projet mentionné dans son portfolio. L’offre exige une attestation d’anglais B2. Elle ajoute la pièce manquante et dépose son dossier. Le jury examine la version déposée, même si Lina met ensuite son portfolio à jour.

## Microservices et répartition

| Responsable | Microservice | Responsabilité | Entités principales |
|---|---|---|---|
| Ela Nour Neily | Portfolio étudiant | Gérer les expériences et les justificatifs réutilisables. | Experience, Justificatif |
| Mehdi Essid | Candidatures | Préparer le brouillon, figer le dossier au dépôt et suivre son état. | Candidature, PieceCandidature |
| Med Yassine Mejri | Présélection | Définir le barème, évaluer les dossiers et valider la liste des candidats retenus. | GrillePreselection, EvaluationDossier |
| Maghrebi Yassine | Épreuves et entretiens | Planifier les sessions, gérer les convocations et enregistrer la décision finale. | SessionEvaluation, Convocation |
| Mokcheh Rahim | Catalogue des opportunités | Publier les offres, définir leurs critères et expliquer leur compatibilité avec le portfolio. | Opportunite, CritereOffre |
| Toute l’équipe | Utilisateurs et accès | Relier les profils aux identités Keycloak et gérer les affiliations aux organismes. | ProfilUtilisateur, Affiliation |

Le Catalogue utilise la recherche sémantique avec *pgvector* pour rapprocher les expériences de l’étudiant des offres. Son indice de compatibilité guide la recherche avant le dépôt. Après le dépôt, le service Présélection calcule un score distinct selon le barème du jury.

## Architecture technique prévue

Le frontend *Angular* utilise *Keycloak* pour la connexion avec OpenID Connect. Il transmet ensuite son jeton JWT à *Spring Cloud Gateway*, point d’entrée des API. La Gateway utilise *Eureka* pour découvrir les instances des microservices *Spring Boot*. *Spring Cloud Config Server* centralise leur configuration.

Chaque microservice possède sa propre base *PostgreSQL* et peut être déployé indépendamment. Les relations entre services utilisent des identifiants externes. Par exemple, Candidatures conserve un offreId et consulte le Catalogue via son API. Le déploiement local prévu utilise *Docker Compose*.

### Exemple de communication au dépôt d’un dossier

1. Angular envoie POST /api/v1/candidatures/{id}/depot à la Gateway.
2. Candidatures consulte le Catalogue et le Portfolio par *REST/OpenFeign* pour vérifier que l’offre est ouverte et récupérer les versions choisies des pièces.
3. Une fois le dossier figé et enregistré, Candidatures publie CandidatureDeposee via *RabbitMQ*. Présélection peut alors récupérer le dossier à évaluer.
4. Lorsque le jury valide sa liste, PreselectionValidee permet au service Épreuves et entretiens de préparer les convocations. Après la dernière épreuve, DecisionFinaleEnregistree permet de mettre à jour le suivi étudiant.

## Organisation prévue du dépôt

text
Backend/                  Projets Spring Boot, un par microservice
Frontend/                 Application Angular
Documentation/
├── Presentation/         Support de soutenance
└── Diagrams/             Architecture globale et diagrammes de classes
README.md                 Présentation du projet et instructions de lancement

Le dossier technique et le catalogue des API sont destinés à Documentation/.

## Lancement local

Pour la validation intermédiaire, chaque responsable doit pouvoir compiler et démarrer son propre projet. Les commandes ci-dessous s’appliquent à un microservice Spring Boot initialisé avec *Maven Wrapper*, depuis le dossier contenant son fichier pom.xml.

*Windows PowerShell*

.\mvnw.cmd clean verify
.\mvnw.cmd spring-boot:run

*Linux ou macOS*

./mvnw clean verify
./mvnw spring-boot:run

Pour le frontend, une fois l’application Angular initialisée, exécuter depuis Frontend/ :

npm install
npm start

Lorsque le fichier Compose de la plateforme sera disponible, lancer la commande suivante depuis son dossier :

docker compose up --build

Les versions requises de Java et Node.js, les paramètres locaux et les URL d’accès seront précisés avec la configuration effective du dépôt.