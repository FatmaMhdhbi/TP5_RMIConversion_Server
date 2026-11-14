# TP 5 – Activité 5-1 : Service de conversion de devise avec Java RMI

**Matière :** Développement d'Applications Réparties – LSI 3
**Faculté des Sciences de Sfax** – Année universitaire 2026 – 2027
**Type :** Individuel

---

## 1. Objectif

Concevoir une application répartie client/serveur avec **Java RMI (Remote Method Invocation)** qui offre un service de conversion d'un montant en **euros vers le dinar**.

Le cycle de vie du service distant couvre quatre phases :
**Définition de l'interface → Implémentation → Enregistrement → Invocation.**


## 2. Travail réalisé

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

## 3. Captures d'écran et explications

### 3.1 Structure du projet serveur
<img width="346" height="673" alt="Capture d&#39;écran 2026-11-13 214201" src="https://github.com/user-attachments/assets/5ef6972b-6892-4a46-a044-a6a8b5211a49" />


Le projet serveur contient le package `rmiService` (interface `IConversion` et classe `ConversionImpl`) et le package `rmiServer` (classe `ConversionServer`). Le dossier `out/artifacts/ConversionClient` contient le fichier `ConversionClient.jar` généré pour le client.

### 3.2 Création du JAR (Artifacts)
<img width="1030" height="472" alt="Capture d&#39;écran 2026-11-13 214615" src="https://github.com/user-attachments/assets/22bbfd12-f786-4760-9788-5af3a262da29" />


Dans *Project Structure → Artifacts*, un artefact de type **JAR** nommé `ConversionClient` a été créé. Dans son *Output Layout*, seule la classe `rmiService/IConversion.class` est incluse, conformément à la consigne (sans `ConversionImpl`).

### 3.3 Structure du projet client
<img width="360" height="660" alt="Capture d&#39;écran 2026-11-13 214239" src="https://github.com/user-attachments/assets/b5bf6880-c381-4dc1-b196-f8d393f32029" />


Le projet client contient le dossier `lib` avec `ConversionClient.jar` (qui expose `rmiService.IConversion`) et le package `rmiClient` avec la classe `ConversionClient`. Le JAR apparaît aussi dans *External Libraries*.

### 3.4 Ajout du JAR aux dépendances du client
<img width="1004" height="437" alt="Capture d&#39;écran 2026-11-13 214558" src="https://github.com/user-attachments/assets/706d31fc-ac67-48aa-a02a-85fb1708b28e" />


Dans *Project Structure → Modules → Dependencies*, `ConversionClient.jar` est ajouté avec la portée **Compile**. Le client peut ainsi utiliser l'interface `IConversion` sans disposer du code du serveur.

### 4.5 Démarrage du serveur RMI
<img width="742" height="171" alt="Capture d&#39;écran 2026-11-13 205329" src="https://github.com/user-attachments/assets/742bd18b-e8ed-4b11-b786-3ae223288593" />



La console affiche le message **« Serveur RMI de conversion démarré... »** : le registre RMI est actif et l'objet `ConversionImpl` est publié (la ligne `ConversionImpl[UnicastServerRef [liveRef: [endpoint:[192.168.1.13:57076]...` montre l'objet distant exporté, avec l'adresse IP et le port sur lesquels il écoute).

### 4.6 Exécution du client
<img width="434" height="223" alt="Capture d&#39;écran 2026-11-13 213926" src="https://github.com/user-attachments/assets/03d86f6d-555b-4ec4-accf-457763f38f8d" />


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
