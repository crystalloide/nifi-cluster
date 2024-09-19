### [Home (README.md)](../README.md)
---

# Custom Processors / Processeurs personnalisés

Consultez le [Guide d'administration NiFi](https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#processor-locations) pour plus de détails sur l'emplacement où placer les custom processor NARs. 

Consultez la section « Issues » plus bas pour voir la discussion sur ce qui a été fait.

Utilisez la commande docker copy pour placer vos fichiers NAR de processeur personnalisés dans le répertoire de bibliothèque automatique, 

qui est */opt/nifi/nifi-current/extensions/*. 

Assurez-vous d'utiliser l'indicateur *--all* pour garantir que le fichier est copié dans tous les conteneurs NiFi.

```
docker compose cp --all <path-to-nar-file> nifi:/opt/nifi/nifi-current/extensions/
```

Notez que pour ces images Docker, le NAR doit avoir été compilé sous Java 1.8.0. 

Actualisez l'interface graphique NiFi dans votre navigateur avant d'essayer d'utiliser le nouveau processeur.

Si vous essayez de charger le processeur **à nouveau**, vous devrez redémarrer le service NiFi, car les chargeurs de classe ne chargeront pas une classe existante.

```
docker compose restart nifi
```

## Text Approval Processor (Processeur d'approbation de texte) :

Il existe un projet maven inclus qui crée un processeur très simple qui ajoute simplement la phrase "APPROVED" à la fin de tout message qu'il voit dans le flux.

Créez le projet avec ``mvn clean package``  

cela produit un fichier NAR *archiver/target/nifi-hindmasj-processors-&lt;version&gt;.nar* 

qui peut ensuite être chargé dans le cluster comme indiqué ci-dessus.


---
### [Home (README.md)](../README.md)
