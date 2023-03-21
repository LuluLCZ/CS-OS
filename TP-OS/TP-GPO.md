On vérifie qu'aucun autre computer est enregistré dans l'AD a part la machine linux précédemment enregistrée.
![[Pasted image 20230321141750.png]]
On met a jour le DNS sur le client W11 avec l'ip de l'AD (10.0.20.100)
![[Pasted image 20230321142657.png]]
On peut maintenant rejoindre l'AD:
![[Pasted image 20230321142751.png]]
succès:
![[Pasted image 20230321142831.png]]
On retrouve le nouveau computer dans l'active directory:
![[Pasted image 20230321142918.png]]
On ajoute deux forets:
![[Pasted image 20230321152652.png]]
Création utilisateur Alice dans OU Rennes:
![[Pasted image 20230321153202.png]]
On active la GPO permettant de refuser l'utilisation du command prompt sur l'OU Saclay:
![[Pasted image 20230321153621.png]]
Creation de Bob dans OU Saclay
![[Pasted image 20230321153906.png]]
On voit bel et bien que le cmd.exe est unauthorized pour les utilisateurs appartenant à l'OU Saclay:
![[Pasted image 20230321154131.png]]
Création de la PSO pour password Settings avec un nb de caractères minimum de 1:
![[Pasted image 20230321154509.png]]
PSO bien créée:
![[Pasted image 20230321154600.png]]
Voici la classe de notre objet SaclayPassword dans l'application ADSI Edit:
![[Pasted image 20230321155037.png]]
`msDS-PasswordSettings`
La PSO s'applique bien au groupe:
![[Pasted image 20230321155150.png]]
À l'aide de la commande Get-ADGroupMember on retrouve bien bob dans le groupe SaclayGroup
![[Pasted image 20230321155437.png]]
## 5
On créer un dossier `C:\Applications` dans l'AD que l'on partage en lecture seule:
![[Pasted image 20230321155731.png]]
On vérifie que le dossier est bien accessible depuis la machine cliente:
![[Pasted image 20230321155838.png]]
On supprime l'inhéritence du C: pour pouvoir autoriser seulement les ordinateurs a acceder a ce dossier et non les utilisateurs du domaine:
![[Pasted image 20230321160009.png]]
On autorise les DomainComputers et les administrators a avoir accès à ce dossier:
![[Pasted image 20230321160618.png]]
On ajoute la GPO Firefox contenant le fichier .msi dans `\\ADDS\Applications`
![[Pasted image 20230321161510.png]]
Une fois créer, on Link la GPO Firefox (clique droit sur DOMAIN.INT et link GPO:
![[Pasted image 20230321163039.png]])

On force la mise a jour de la GPO sur le client AD:
![[Pasted image 20230321162920.png]]
On redemarre et on castate que la GPO à été executée car Firefox est installé sur l'ordinateur
![[Pasted image 20230321163111.png]]
On créer une nouvelle GPO sur le groupe Saclay permettant aux utilisateurs de venir chercher une nouvelle installation disponible dans le domain (published et non assigned, le packet sera publié et donc disponible dans le réseau et non installé au redémarrage)
Alice ne l'a pas car elle est dans l'OU Rennes et non Saclay:
![[Pasted image 20230321163517.png]]
Avec Bob, on le retrouve bien
![[Pasted image 20230321163642.png]]
Après avoir modifié les permissions d'accès au dossier `\\ADDS\Applications` et rajouté `SaclayGroup` en authorized, on installe chrome:
![[Pasted image 20230321163901.png]]
