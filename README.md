# Linux_short Day2
 
 **GUEST ADDitions** le plus gros interet pour nous :  
 - presse-papier partagé  
 - graphique amelioré (peut utiliser les drivers du PC, et avoir une full resolution)  
 
**Dynamic Kernel Module Support (DKMS)** est un programme/framework qui permet de générer des modules du noyau Linux dont les sources résident généralement en dehors de l'arbre des sources du noyau. Le concept est de reconstruire automatiquement les modules DKMS lorsqu'un nouveau noyau est installé.  

**uname (diminutif pour unix name)** est une commande Unix qui affiche les informations système sur la machine sur laquelle elle est exécutée.  
*Il est possible qu'on n'ait pas la mm version* www.kernel.org   

**Linux Headers**, ce sont des bibliothèques nécessaires pour développer et compiler des programmes.  

check if installed and all ok :
```
lsmod | grep vbox
sudo modinfo vboxguest
```
If issue:
```
sudo mount /dev/cdrom /mnt
sudo sh /mnt/VBoxLinuxAdditions.run
Reboot
```

-----------------------------------------------

# Command Line

## CLI vs. GUI

```
hawai@nitro5:~$

hawai = who
nitro5 = machine 
~ = where
$ user  # root

```
> date = when

```
whoami
who -> tty = teletype = console
dev/pts = pseudo-terminal
dev/pts/ptmx = the file /dev/ptmx (the pseudoterminal multiplexor device) 
if we do ls -l on /dev/tty* we can see a c

```

```
The file type is one of the following characters:
    -  regular file
    b  block special file
    c  character special file
    C  high performance ("contiguous data") file
    d  directory
    D  door (Solaris 2.5 and up)
    l  symbolic link
    M  off-line ("migrated") file (Cray DMF)
    n  network special file (HP-UX)
    p  FIFO (named pipe)
    P  port (Solaris 10 and up)
    s  socket
    ?  some other file type

```
    
> type / which / file

**ELF** : Executable and Linkable Format  
**interpreter /lib64/ld-linux-x86-64.so.2**: You have invoked ‘ld.so’, the helper program for shared library executables.  


man builtins  


**Everything is a file**  

https://en.wikipedia.org/wiki/Unix_filesystem#Conventional_directory_layout

man hier  

example : cd  

man  


```
echo (bash command/script)  

pwd  
ls  

cat  
tail/head  

touch  
mkdir  
cp  
mv  
ln -s  
rm  

```

-------

grep --> regex
find  


