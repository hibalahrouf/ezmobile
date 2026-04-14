# Exercice 1 — ezmobile

## Niveau
Débutant

## Objectif
Cette première épreuve consiste à réaliser une **analyse statique simple** d'une application Android afin d'identifier la logique de validation d'un secret.  
Le but est de retrouver le **flag** **sans modifier l'application** et **sans exécution intrusive**, uniquement en étudiant le contenu de l'APK.

---

## Démarche suivie

### 1. Décompilation de l'APK
L'APK a d'abord été ouvert avec **JADX** afin d'explorer :
- les classes Java décompilées ;
- les ressources Android embarquées ;
- les fichiers XML présents dans le dossier `res/values/`.

Cette étape permet d'obtenir une vue claire de la structure de l'application et de repérer rapidement les éléments intéressants.

> **Figure 1** — Vue de l'interface JADX avec `MainActivity` ouverte et la fonction `FlagCheck` visible.  
> ![JADX MainActivity](image/WhatsApp_Image_2026-04-08_at_14_37_23__4_.jpeg)

---

### 2. Exploration des classes principales
En parcourant les classes de l'application, la classe principale **`MainActivity`** a été identifiée comme point d'entrée logique du challenge.

Dans cette classe, une fonction nommée **`FlagCheck(View view)`** attire immédiatement l'attention, car son nom indique clairement qu'elle est responsable de la vérification du flag.

La méthode observée est la suivante :

```java
public void FlagCheck(View view) {
    if (new String(Base64.decode(getString(R.string.Flag), 0))
            .equals(((EditText) findViewById(R.id.input)).getText().toString())) {
        Toast.makeText(this, "Congrats you got the flag!!!", 0).show();
    } else {
        Toast.makeText(this, "Try Harder ;)", 0).show();
    }
}
```

---

### 3. Analyse de la logique de validation

L'analyse de cette fonction montre le mécanisme exact de vérification :

- l'application récupère une ressource Android appelée `R.string.Flag` ;
- cette valeur n'est pas utilisée directement ;
- elle est d'abord décodée avec `Base64.decode(...)` ;
- le résultat est converti en chaîne de caractères ;
- cette chaîne décodée est ensuite comparée au texte saisi par l'utilisateur dans le champ `input`.

Autrement dit, la condition de succès est :

```
Base64.decode(getString(R.string.Flag), 0) == saisie_utilisateur
```

Cela signifie que le flag recherché n'est pas stocké en clair dans l'interface, mais dans les ressources de l'application, sous une forme **encodée en Base64**.

---

### 4. Localisation de la donnée comparée

En explorant les ressources Android, notamment le fichier :

```
res/values/strings.xml
```

on trouve l'entrée suivante :

```xml
<string name="Flag">UFdOU0VDe3czbHBfbjJAaCFuZ19TcDNaTRsX0p1c3RfNF9GbDRnXyFuXzdoM19zN3Ihbmc1X3htbF9mIWwzfQ</string>
```

> **Figure 2** — Fichier `strings.xml` avec la ressource `Flag` mise en évidence.  
> ![strings.xml](image/WhatsApp_Image_2026-04-08_at_14_37_24__5_.jpeg)

Cette chaîne correspond à la valeur référencée par `R.string.Flag` dans la fonction `FlagCheck()`.

Par ailleurs, le fichier de ressources généré `R.java` confirme également l'existence de cette ressource :

```java
public static int Flag = 0x7f0f0000;
```

> **Figure 3** — Fichier `R.java` généré par JADX, montrant l'identifiant de la ressource `Flag`.  
> ![R.java](WhatsApp_Image_2026-04-08_at_14_37_23__3_.jpeg)

Cela valide que la donnée comparée provient bien des ressources Android et non d'une valeur construite dynamiquement ailleurs dans le code.

---

### 5. Décodage du flag

La valeur trouvée dans `strings.xml` a ensuite été décodée avec **CyberChef** en utilisant l'opération :

```
From Base64
```

> **Figure 4** — Décodage Base64 dans CyberChef, avec le flag en sortie.  
> ![CyberChef décodage](image/WhatsApp_Image_2026-04-08_at_14_37_24__4_.jpeg)

Le résultat obtenu est :

```
PWNSEC{w3lp_n07h!ng_Sp3ci4l_Just_4_Fl4g_In_7h3_s7r!ng5_xml_f!l3}
```

---

## Flag

```
PWNSEC{w3lp_n07h!ng_Sp3ci4l_Just_4_Fl4g_In_7h3_s7r!ng5_xml_f!l3}
```

---

## Vérification

Une saisie manuelle du flag dans l'application affiche le message :

```
Congrats you got the flag!!!
```

> **Figure 5** — Confirmation visuelle dans l'émulateur Android après saisie du flag.  
> ![Confirmation Android](image/WhatsApp_Image_2026-04-08_at_14_37_24__3_.jpeg)

Cela confirme que la valeur décodée correspond bien au flag attendu par l'application.

---

## Rôle des ressources Android dans ce challenge

Les ressources Android jouent un rôle central dans cette épreuve.

Dans une application Android, les ressources servent à stocker différents éléments séparément du code source, par exemple :

- les chaînes de caractères ;
- les couleurs ;
- les dimensions ;
- les mises en page ;
- les identifiants utilisés par l'interface.

Dans ce challenge, la ressource `string` nommée `Flag` contient directement la donnée utilisée pour la vérification.  
Le développeur a simplement choisi de ne pas la stocker en clair, mais de l'**encoder en Base64**.

Ainsi, les ressources Android ne sont pas seulement utiles à l'interface graphique : elles peuvent aussi contenir des données sensibles, des indices, voire directement le secret recherché.

---

## Justification synthétique

Le flag a été retrouvé par **analyse statique** de l'APK.  
L'exploration de `MainActivity` a permis d'identifier la fonction `FlagCheck(View view)`, dans laquelle la saisie utilisateur est comparée à une valeur issue de `R.string.Flag` après décodage Base64.  
La ressource correspondante a été localisée dans `res/values/strings.xml`, puis décodée avec CyberChef, ce qui a permis d'obtenir le flag final **sans patcher l'application ni utiliser d'instrumentation dynamique**.

---

## Conclusion

Cette épreuve illustre un cas classique de reverse engineering Android de niveau débutant.  
La logique de validation était directement visible dans le code décompilé, et le secret était stocké dans les ressources de l'application sous forme encodée.  
Une simple analyse statique a donc suffi pour :

- identifier la fonction de vérification ;
- localiser la donnée comparée ;
- comprendre le rôle des ressources Android ;
- extraire et valider le flag.

---

## Résumé final

| Élément                    | Valeur                                                                |
|----------------------------|-----------------------------------------------------------------------|
| Classe principale analysée | `MainActivity`                                                        |
| Fonction de vérification   | `FlagCheck(View view)`                                                |
| Donnée comparée            | `R.string.Flag`                                                       |
| Emplacement de la donnée   | `res/values/strings.xml`                                              |
| Transformation appliquée   | Décodage Base64                                                       |
| **Flag obtenu**            | `PWNSEC{w3lp_n07h!ng_Sp3ci4l_Just_4_Fl4g_In_7h3_s7r!ng5_xml_f!l3}` |
