# Manipulation des Fichiers HDFS - Solution Docker

## Configuration de l'Environnement

### Prérequis
- Docker installé sur votre système
- Docker Compose installé sur votre système

### Configuration Docker

Nous utiliserons les images Docker Hadoop de Big Data Europe pour une configuration plus stable. Créez les fichiers suivants :

1. Créez un fichier `docker-compose.yml` :
```yaml
version: '3'

services:
  namenode:
    image: bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8
    container_name: namenode
    restart: always
    ports:
      - 9870:9870
      - 9000:9000
    volumes:
      - hadoop_namenode:/hadoop/dfs/name
    environment:
      - CLUSTER_NAME=test
      - CORE_CONF_fs_defaultFS=hdfs://namenode:9000
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9870"]
      interval: 30s
      timeout: 10s
      retries: 3

  datanode:
    image: bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8
    container_name: datanode
    restart: always
    volumes:
      - hadoop_datanode:/hadoop/dfs/data
    environment:
      - CORE_CONF_fs_defaultFS=hdfs://namenode:9000
    depends_on:
      - namenode

  resourcemanager:
    image: bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8
    container_name: resourcemanager
    restart: always
    ports:
      - 8088:8088
    environment:
      - CORE_CONF_fs_defaultFS=hdfs://namenode:9000
    depends_on:
      - namenode
      - datanode

  nodemanager:
    image: bde2020/hadoop-nodemanager:2.0.0-hadoop3.2.1-java8
    container_name: nodemanager
    restart: always
    environment:
      - CORE_CONF_fs_defaultFS=hdfs://namenode:9000
    depends_on:
      - namenode
      - datanode
      - resourcemanager

volumes:
  hadoop_namenode:
  hadoop_datanode:
```

## Implémentation Étape par Étape avec Résultats

### 1. Démarrage du Cluster Hadoop
```bash
# Démarrer tous les services
sudo docker-compose up -d

# Sortie :
Creating network "hdfs-files-manipuation_default" with the default driver
Creating volume "hdfs-files-manipuation_hadoop_namenode" with default driver
Creating volume "hdfs-files-manipuation_hadoop_datanode" with default driver
Creating namenode        ... done
Creating datanode        ... done
Creating resourcemanager ... done
Creating nodemanager     ... done

# Vérifier que les conteneurs sont en cours d'exécution
sudo docker ps

# Sortie :
CONTAINER ID   IMAGE                                                    COMMAND                  CREATED          STATUS                            PORTS                                                                                  NAMES
182faded097c   bde2020/hadoop-nodemanager:2.0.0-hadoop3.2.1-java8     "/entrypoint.sh /run…"   5 seconds ago    Up 4 seconds (health: starting)   8042/tcp                                                                             nodemanager
ebc2a8064e34   bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8 "/entrypoint.sh /run…"   5 seconds ago    Up 4 seconds (health: starting)   0.0.0.0:8088->8088/tcp, :::8088->8088/tcp                                           resourcemanager
7e0e7cef9014   bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8        "/entrypoint.sh /run…"   6 seconds ago    Up 4 seconds (health: starting)   9864/tcp                                                                             datanode
e3d0feb226f8   bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8        "/entrypoint.sh /run…"   6 seconds ago    Up 5 seconds (health: starting)   0.0.0.0:9000->9000/tcp, :::9000->9000/tcp, 0.0.0.0:9870->9870/tcp, :::9870->9870/tcp namenode
```

### 2. Création de la Structure des Répertoires

```bash
# Créer tous les répertoires
sudo docker exec -it namenode bash -c "hdfs dfs -mkdir /BDDC && hdfs dfs -mkdir /BDDC/JAVA /BDDC/CPP && hdfs dfs -mkdir /BDDC/JAVA/Cours /BDDC/JAVA/TPs /BDDC/CPP/Cours /BDDC/CPP/TPs"

# Vérifier la structure des répertoires
sudo docker exec -it namenode bash -c "hdfs dfs -ls -R /BDDC"

# Sortie :
drwxr-xr-x   - root supergroup          0 2025-01-13 12:09 /BDDC/CPP
drwxr-xr-x   - root supergroup          0 2025-01-13 12:09 /BDDC/CPP/Cours
drwxr-xr-x   - root supergroup          0 2025-01-13 12:09 /BDDC/CPP/TPs
drwxr-xr-x   - root supergroup          0 2025-01-13 12:09 /BDDC/JAVA
drwxr-xr-x   - root supergroup          0 2025-01-13 12:09 /BDDC/JAVA/Cours
drwxr-xr-x   - root supergroup          0 2025-01-13 12:09 /BDDC/JAVA/TPs
```

### 3. Création et Manipulation des Fichiers

```bash
# Créer les fichiers de cours CPP
sudo docker exec -it namenode bash -c 'echo "Content for CPP Course 1" | hdfs dfs -put - /BDDC/CPP/Cours/CoursCPP1 && echo "Content for CPP Course 2" | hdfs dfs -put - /BDDC/CPP/Cours/CoursCPP2 && echo "Content for CPP Course 3" | hdfs dfs -put - /BDDC/CPP/Cours/CoursCPP3'

# Afficher le contenu des fichiers
sudo docker exec -it namenode bash -c 'echo "=== CoursCPP1 ===" && hdfs dfs -cat /BDDC/CPP/Cours/CoursCPP1 && echo -e "\n=== CoursCPP2 ===" && hdfs dfs -cat /BDDC/CPP/Cours/CoursCPP2 && echo -e "\n=== CoursCPP3 ===" && hdfs dfs -cat /BDDC/CPP/Cours/CoursCPP3'

# Sortie :
=== CoursCPP1 ===
Content for CPP Course 1

=== CoursCPP2 ===
Content for CPP Course 2

=== CoursCPP3 ===
Content for CPP Course 3

# Copier les fichiers vers JAVA/Cours
sudo docker exec -it namenode bash -c 'hdfs dfs -cp /BDDC/CPP/Cours/CoursCPP1 /BDDC/CPP/Cours/CoursCPP2 /BDDC/CPP/Cours/CoursCPP3 /BDDC/JAVA/Cours/'

# Supprimer et renommer les fichiers dans JAVA/Cours
sudo docker exec -it namenode bash -c 'hdfs dfs -rm /BDDC/JAVA/Cours/CoursCPP3 && hdfs dfs -mv /BDDC/JAVA/Cours/CoursCPP1 /BDDC/JAVA/Cours/CoursJAVA1 && hdfs dfs -mv /BDDC/JAVA/Cours/CoursCPP2 /BDDC/JAVA/Cours/CoursJAVA2'

# Sortie :
Deleted /BDDC/JAVA/Cours/CoursCPP3
```

### 4. Travail avec les Fichiers Locaux

```bash
# Créer et remplir les fichiers locaux
sudo docker exec -it namenode bash -c 'cd /tmp && echo "TP1 CPP Content" > TP1CPP && echo "TP2 CPP Content" > TP2CPP && echo "TP1 JAVA Content" > TP1JAVA && echo "TP2 JAVA Content" > TP2JAVA && echo "TP3 JAVA Content" > TP3JAVA'

# Copier les fichiers vers HDFS
sudo docker exec -it namenode bash -c 'cd /tmp && hdfs dfs -put TP1CPP TP2CPP /BDDC/CPP/TPs/ && hdfs dfs -put TP1JAVA TP2JAVA /BDDC/JAVA/TPs/'
```

### 5. Opérations Finales

```bash
# Afficher la structure complète du répertoire BDDC
sudo docker exec -it namenode bash -c 'hdfs dfs -ls -R /BDDC'

# Sortie :
drwxr-xr-x   - root supergroup          0 2025-01-13 12:09 /BDDC/CPP
drwxr-xr-x   - root supergroup          0 2025-01-13 12:10 /BDDC/CPP/Cours
-rw-r--r--   3 root supergroup         25 2025-01-13 12:10 /BDDC/CPP/Cours/CoursCPP1
-rw-r--r--   3 root supergroup         25 2025-01-13 12:10 /BDDC/CPP/Cours/CoursCPP2
-rw-r--r--   3 root supergroup         25 2025-01-13 12:10 /BDDC/CPP/Cours/CoursCPP3
drwxr-xr-x   - root supergroup          0 2025-01-13 12:10 /BDDC/CPP/TPs
-rw-r--r--   3 root supergroup         16 2025-01-13 12:10 /BDDC/CPP/TPs/TP1CPP
-rw-r--r--   3 root supergroup         16 2025-01-13 12:10 /BDDC/CPP/TPs/TP2CPP
drwxr-xr-x   - root supergroup          0 2025-01-13 12:09 /BDDC/JAVA
drwxr-xr-x   - root supergroup          0 2025-01-13 12:10 /BDDC/JAVA/Cours
-rw-r--r--   3 root supergroup         25 2025-01-13 12:10 /BDDC/JAVA/Cours/CoursJAVA1
-rw-r--r--   3 root supergroup         25 2025-01-13 12:10 /BDDC/JAVA/Cours/CoursJAVA2
drwxr-xr-x   - root supergroup          0 2025-01-13 12:10 /BDDC/JAVA/TPs
-rw-r--r--   3 root supergroup         17 2025-01-13 12:10 /BDDC/JAVA/TPs/TP1JAVA
-rw-r--r--   3 root supergroup         17 2025-01-13 12:10 /BDDC/JAVA/TPs/TP2JAVA

# Supprimer des fichiers et répertoires spécifiques
sudo docker exec -it namenode bash -c 'hdfs dfs -rm /BDDC/CPP/TPs/TP1CPP && hdfs dfs -rm -r /BDDC/JAVA'

# Sortie :
Deleted /BDDC/CPP/TPs/TP1CPP
Deleted /BDDC/JAVA

# État final
sudo docker exec -it namenode bash -c 'hdfs dfs -ls -R /BDDC'

# Sortie :
drwxr-xr-x   - root supergroup          0 2025-01-13 12:09 /BDDC/CPP
drwxr-xr-x   - root supergroup          0 2025-01-13 12:10 /BDDC/CPP/Cours
-rw-r--r--   3 root supergroup         25 2025-01-13 12:10 /BDDC/CPP/Cours/CoursCPP1
-rw-r--r--   3 root supergroup         25 2025-01-13 12:10 /BDDC/CPP/Cours/CoursCPP2
-rw-r--r--   3 root supergroup         25 2025-01-13 12:10 /BDDC/CPP/Cours/CoursCPP3
drwxr-xr-x   - root supergroup          0 2025-01-13 12:11 /BDDC/CPP/TPs
-rw-r--r--   3 root supergroup         16 2025-01-13 12:10 /BDDC/CPP/TPs/TP2CPP
```

## Nettoyage

Pour arrêter et supprimer le cluster Hadoop :

```bash
# Arrêter les conteneurs et supprimer les volumes
sudo docker-compose down -v

# Sortie :
Removing datanode        ... done
Removing nodemanager     ... done
Removing resourcemanager ... done
Removing namenode        ... done
Removing network hdfs-files-manipuation_default
Removing volume hdfs-files-manipuation_hadoop_namenode
Removing volume hdfs-files-manipuation_hadoop_datanode
```

## Notes
- Les interfaces web sont accessibles aux adresses suivantes :
  - Interface NameNode : http://localhost:9870
  - Interface Resource Manager : http://localhost:8088
- Toutes les opérations HDFS sont effectuées à l'intérieur du conteneur namenode
- La configuration utilise Hadoop 3.2.1 avec Java 8
- Les permissions HDFS sont activées par défaut dans cette configuration
``` 