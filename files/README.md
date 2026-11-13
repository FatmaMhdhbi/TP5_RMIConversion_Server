# TP 5 – Activité 5-1 : Service de conversion de devise avec Java RMI

**Matière :** Développement d'Applications Réparties – LSI 3
**Faculté des Sciences de Sfax** – Année universitaire 2026 – 2027
**Type :** Individuel

---

## 1. Objectif

Concevoir une application répartie client/serveur avec **Java RMI (Remote Method Invocation)** qui offre un service de conversion d'un montant en **euros vers le dinar**.

Le cycle de vie du service distant couvre quatre phases :
**Définition de l'interface → Implémentation → Enregistrement → Invocation.**

## 2. Architecture du projet

```
votreNomPrenom_LSI3_DevAppReparties/
├── TP5_RMIConversion_Server/
│   └── src/main/java/
│       ├── rmiService/
│       │   ├── IConversion.java       (interface distante)
│       │   └── ConversionImpl.java    (implémentation du service)
│       └── rmiServer/
│           └── ConversionServer.java  (lancement du serveur + registre)
├── tP5_RMIConversion_Client/
│   ├── lib/ConversionClient.jar       (contient uniquement IConversion)
│   └── src/main/java/rmiClient/
│       └── ConversionClient.java      (application cliente)
└── README.md
```

## 3. Travail réalisé

### Étape 1 – Préparation de l'espace de travail
Création du répertoire `votreNomPrenom_LSI3_DevAppReparties` qui contient les deux projets (serveur et client).

### Étape 2 – Développement du serveur

**a) Interface distante `IConversion`** (package `rmiService`)
Elle étend `java.rmi.Remote` et déclare la méthode distante `convertirMontant`, qui lève `RemoteException` (obligatoire pour toute méthode distante, car la communication réseau peut échouer).

```java
public interface IConversion extends Remote {
    double convertirMontant(double mt) throws RemoteException;
}
```

**b) Implémentation `ConversionImpl`** (package `rmiService`)
La classe implémente `IConversion` et étend `UnicastRemoteObject`, ce qui permet d'exporter automatiquement l'objet pour qu'il soit accessible à distance. La méthode `convertirMontant` applique le taux de conversion euro → dinar au montant reçu (taux fixe défini dans le code).

**c) Application serveur `ConversionServer`** (package `rmiServer`)
Elle réalise trois opérations :
1. activer l'annuaire **RMIRegistry** (`LocateRegistry.createRegistry(1099)`) ;
2. créer une instance de `ConversionImpl` ;
3. publier la référence de l'objet dans l'annuaire (`Naming.rebind(...)`) sous un nom connu des clients.

### Étape 3 – Développement du client

1. **Export de l'interface :** depuis le projet serveur, on crée un artefact JAR (`ConversionClient.jar`) qui ne contient **que** `rmiService/IConversion.class`. L'implémentation `ConversionImpl` n'est volontairement pas incluse : le client n'a besoin que du *contrat* (l'interface), jamais du code du serveur.
2. **Import du JAR :** `ConversionClient.jar` est ajouté au classpath du projet client (dossier `lib` + dépendance du module).
3. **Application `ConversionClient`** (package `rmiClient`) :
   - recherche la référence distante dans le RMIRegistry (`Naming.lookup(...)`) ;
   - la convertit vers le type `IConversion` ;
   - invoque `convertirMontant(500)` et affiche le résultat.

### Étape 4 – Exécution
Le serveur est lancé en premier, puis le client.

---

## 4. Captures d'écran et explications

### 4.1 Structure du projet serveur
![Structure du serveur](images/03_structure_serveur.png)

Le projet serveur contient le package `rmiService` (interface `IConversion` et classe `ConversionImpl`) et le package `rmiServer` (classe `ConversionServer`). Le dossier `out/artifacts/ConversionClient` contient le fichier `ConversionClient.jar` généré pour le client.

### 4.2 Création du JAR (Artifacts)
![Configuration de l'artefact JAR](images/06_artifact_jar.png)

Dans *Project Structure → Artifacts*, un artefact de type **JAR** nommé `ConversionClient` a été créé. Dans son *Output Layout*, seule la classe `rmiService/IConversion.class` est incluse, conformément à la consigne (sans `ConversionImpl`).

### 4.3 Structure du projet client
![Structure du client](images/04_structure_client.png)

Le projet client contient le dossier `lib` avec `ConversionClient.jar` (qui expose `rmiService.IConversion`) et le package `rmiClient` avec la classe `ConversionClient`. Le JAR apparaît aussi dans *External Libraries*.

### 4.4 Ajout du JAR aux dépendances du client
![Dépendance du client](images/05_dependance_client.png)

Dans *Project Structure → Modules → Dependencies*, `ConversionClient.jar` est ajouté avec la portée **Compile**. Le client peut ainsi utiliser l'interface `IConversion` sans disposer du code du serveur.

### 4.5 Démarrage du serveur RMI
![Serveur démarré](images/01_serveur_demarre.png)

La console affiche le message **« Serveur RMI de conversion démarré... »** : le registre RMI est actif et l'objet `ConversionImpl` est publié (la ligne `ConversionImpl[UnicastServerRef [liveRef: [endpoint:[192.168.1.13:57076]...` montre l'objet distant exporté, avec l'adresse IP et le port sur lesquels il écoute).

### 4.6 Exécution du client
![Résultat du client](images/02_resultat_client.png)

Le client affiche **« Résultat de la conversion : 600.0 »** pour l'appel `convertirMontant(500)` et se termine avec le code `0` (exécution sans erreur). Cela confirme que l'appel de méthode a bien été exécuté sur l'objet distant, côté serveur (500 × 1,2 = 600).

---

## 5. Synthèse

Cette activité m'a permis de mettre en œuvre une communication client/serveur avec **Java RMI** :

- **Interface distante** (`IConversion`) : le contrat partagé entre client et serveur ; elle étend `Remote` et ses méthodes lèvent `RemoteException`.
- **Implémentation** (`ConversionImpl`) : elle reste côté serveur ; l'objet est exporté via `UnicastRemoteObject`.
- **Enregistrement** : le serveur crée le **RMIRegistry** et y publie la référence de l'objet sous un nom (`rebind`).
- **Invocation** : le client fait un `lookup` dans l'annuaire, obtient un **stub** (proxy) et appelle la méthode comme s'il s'agissait d'un objet local ; RMI gère la sérialisation et la communication réseau.
- **Séparation client/serveur** : grâce au JAR contenant uniquement l'interface, le client ne dépend pas de l'implémentation.

**Ordre d'exécution :** toujours démarrer le serveur (création du registre) avant le client, sinon le `lookup` échoue (`ConnectException`).

**Résultat :** l'application fonctionne correctement ; la conversion de 500 € renvoie 600.0 (taux fixe utilisé dans l'implémentation).
