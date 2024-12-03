# Installation et utilisation du package `ros2_kinect2` avec Kinect V2 et ROS 2 Humble

## 1. Prérequis

Avant de commencer, assurez-vous d’avoir un environnement ROS 2 Humble fonctionnel. Si ce n’est pas le cas, installez ROS 2 en suivant les [instructions officielles](https://docs.ros.org/en/humble/Installation.html).

### Dépendances nécessaires :
- **`argparse` pour C++**
- [**`libfreenect2`**](https://github.com/maxxlef/Kinect2/blob/main/README.md#installation-et-configuration-de-libfreenect2-sur-ubuntu-2204): le driver de la Kinect V2

## 2. Préparation de l’espace de travail

Placez-vous dans le répertoire de travail ROS 2 et créez un workspace si ce n’est pas déjà fait.
Pour apprendre à faire un espace de travail voir la [vidéo Youtube](https://youtu.be/Y_SyQXTL2XU?t=16).

```bash
cd ~/dev_ws/src
```

### Cloner le dépôt `argparse` pour C++ :
Le package `ros2_kinect2` nécessite la bibliothèque `argparse` pour C++.

```bash
git clone https://github.com/p-ranav/argparse.git
```

### Cloner le dépôt `ros2_kinect2` :
Clonez le dépôt principal pour connecter votre Kinect V2 à ROS 2.

```bash
git clone https://github.com/threeal/ros2_kinect2.git
```

## 3. Installation et configuration

### 3.1. Modifier le fichier `CMakeLists.txt`
Dans le dossier `ros2_kinect2`, ouvrez et modifiez le fichier `CMakeLists.txt` pour vous assurer que toutes les dépendances sont correctement incluses.
Il faut le remplacer par : [CMakeList.txt](https://github.com/maxxlef/Kinect2/blob/main/CMakeLists.txt)

### 3.2. Trouver le bon chemin pour `libfreenect2`
Vous devez indiquer le chemin vers la bibliothèque `libfreenect2` utilisée par Kinect. Pour trouver son emplacement, utilisez la commande suivante :

```bash
sudo find / -name "libfreenect2.so*"
```

Une fois trouvé, assurez-vous que le chemin correct est utilisé dans votre configuration.

### 3.3. Modifier le fichier `.bashrc` pour le chemin des librairies
Ouvrez votre fichier `.bashrc` (ou `.zshrc` si vous utilisez zsh) et ajoutez la ligne suivante pour configurer le chemin des bibliothèques dynamiques :

```bash
nano ~/.bashrc
```

Ajoutez cette ligne à la fin du fichier :

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib:/home/maxime/libfreenect2/build/lib
```

Enregistrez et fermez l’éditeur (avec `Ctrl+X` pour quitter, puis `Y` pour enregistrer).

Rechargez la configuration de votre shell pour appliquer les modifications :

```bash
source ~/.bashrc
```

(Chaque nouvelle session de terminal nécessitera cette commande, ou utilisez `source ~/.bashrc` après chaque redémarrage du terminal).

## 4. Construction du package ROS 2

Une fois les modifications terminées, compilez le package avec `colcon` :

```bash
cd ~/dev_ws
colcon build
```

## 5. Sourcer l’espace de travail

Une fois la compilation terminée, assurez-vous que votre espace de travail est sourcé pour que ROS 2 reconnaisse le package :

```bash
source install/setup.bash
source ~/.bashrc
```

## 6. Utilisation du package

### 6.1. Lancer le nœud Kinect
Une fois le package construit et l’espace de travail sourcé, lancez le nœud principal qui se connecte à la Kinect et publie les flux vidéo et de profondeur sur les topics ROS 2 :

```bash
ros2 run ros2_kinect2 kinect2_node
```

### 6.2. Vérification des topics disponibles
Dans un nouveau terminal (ne fermez pas celui avec le nœud en cours d’exécution), vous pouvez vérifier les topics publiés par le nœud Kinect :

```bash
ros2 topic list
```

Vous devriez voir des topics comme `/kinect2/color/image_raw` ou `/kinect2/depth/image_raw`.

### 6.3. Ajouter un `fixed frame` pour la transformation
Si le frame de référence (`fixed frame`) n'est pas défini, ajoutez-le avec la commande suivante pour la transformation :

```bash
ros2 run tf2_ros static_transform_publisher 0 0 0 0 0 0 /kinect2_link /camera_link
```

### 6.4. Visualiser les données dans RViz2
Pour visualiser les flux vidéo et de profondeur dans RViz2, lancez RViz2 et configurez l'affichage :

```bash
rviz2
```

- Ajoutez un affichage de type `Image` pour afficher la vidéo en temps réel. Sélectionnez le topic `/kinect2/color/image_raw`.

## 7. Tester les topics

Vous pouvez tester un topic spécifique pour voir les données qu’il publie :

```bash
ros2 topic echo /kinect2/color/image_raw
```

---

## Résumé des commandes

| Action                                    | Commande                                                                                   |
|-------------------------------------------|--------------------------------------------------------------------------------------------|
| Cloner `argparse`                         | `git clone https://github.com/p-ranav/argparse.git`                                          |
| Cloner `ros2_kinect2`                     | `git clone https://github.com/threeal/ros2_kinect2.git`                                     |
| Trouver `libfreenect2.so`                 | `sudo find / -name "libfreenect2.so*"`                                                     |
| Modifier `.bashrc`                        | `nano ~/.bashrc`                                                                            |
| Construire l’espace de travail            | `colcon build`                                                                              |
| Sourcer l’espace de travail               | `source install/setup.bash` et `source ~/.bashrc`                                           |
| Lancer le nœud principal                  | `ros2 run ros2_kinect2 kinect2_node`                                                       |
| Vérifier les topics                       | `ros2 topic list`                                                                           |
| Ajouter un `fixed frame`                  | `ros2 run tf2_ros static_transform_publisher 0 0 0 0 0 0 /kinect2_link /camera_link`        |
| Visualiser dans RViz2                    | `rviz2`                                                                                   |



# Installation et configuration de libfreenect2 sur Ubuntu 22.04

Ce guide explique comment configurer **libfreenect2** sur Ubuntu 22.04 pour l'utiliser avec un Kinect Xbox One (Kinect v2). Cette configuration est adaptée à un système avec une **NVIDIA RTX 4070** et un processeur **AMD Ryzen**.

## 1. Prérequis système

Assurez-vous d'avoir installé les pilotes NVIDIA et CUDA pour tirer pleinement parti de votre GPU. Ensuite, installez les outils de développement requis :

```bash
sudo apt-get update
sudo apt-get install build-essential cmake pkg-config libusb-1.0-0-dev libturbojpeg0-dev libglfw3-dev
```

## 2. Installation de libfreenect2

### Clonez le dépôt Git et construisez-le :

```bash
git clone https://github.com/OpenKinect/libfreenect2.git
cd libfreenect2
```

Si nécessaire, installez les dépendances pour les systèmes plus anciens :

```bash
cd depends
./download_debs_trusty.sh
```

Ensuite, revenez dans le répertoire racine du projet et construisez le projet :

```bash
cd ..
mkdir build && cd build
cmake ..
make
sudo make install
```

## 3. Configurer les règles udev pour Kinect

Configurez les règles udev pour permettre l'accès au périphérique Kinect sans avoir besoin de `sudo` :

```bash
sudo cp ../platform/linux/udev/90-kinect2.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Rebranchez votre périphérique Kinect.

## 4. Tester le Kinect avec Protonect

Pour vérifier l'installation et voir si le Kinect est détecté, exécutez le programme `Protonect` depuis le répertoire `build` :

```bash
./bin/Protonect
```

## 5. Dépannage

Si le Kinect n'est pas détecté :

1. Débranchez et rebranchez le périphérique Kinect.
2. Vérifiez que les règles udev sont correctement appliquées en relançant :
   ```bash
   sudo cp ../platform/linux/udev/90-kinect2.rules /etc/udev/rules.d/
   sudo udevadm control --reload-rules
   sudo udevadm trigger
   ```

Une fois cela effectué, essayez de lancer `Protonect` à nouveau.

## Ressources supplémentaires

- [Dépôt GitHub de libfreenect2](https://github.com/OpenKinect/libfreenect2)
