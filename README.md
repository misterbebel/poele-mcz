/***************************************************************************
      Tutoriel d'installation sur environnement Raspbian (debian pour RPI)
***************************************************************************/
A noter : $ -> utilisateur (pi par défault sous Raspbian)  # -> administrateur (compte root)
prérequis : Avant toute installation lancer la commande :
# apt-get update
# apt-get upgrade
/**********************************************
       Packages et AppliMCZ
**********************************************/       
- Installation des packages pour le developpement:
# apt-get install build-essential

- Installation du parser XML:
# apt-get install libmxml-dev

- Installation de Wiring PI pour envoyer les donnees RF:
# git clone git://git.drogon.net/wiringPi
# cd wiringPi
# ./build

- Creer un repertoire de developement dans /home/<user> (au choix) bebel et poele
$ cd /home/<user>/<dev repertoire>
$ git clone https://github.com/KerberRF/APPLI_MCZ.git

- Compilation de l'application avec GCC.
A lancer dans le répertoire de l'application bien sur.
# make

- liens utiles: 
git clone git://git.drogon.net/wiringPi
https://github.com/ninjablocks/433Utils
https://github.com/landru29/chacon-rpi

/************************************************
                     NOTES:
************************************************/                     
Les données sont envoyées sur un lien GPIO (pin 22) connecté sur un emetteur RF433Mhz:
http://www.amazon.fr/Neuftech%C2%AE-433mhz-transmetteur-r%C3%A9cepteur-Arduino/dp/B00NIBI7IK/ref=sr_1_2?ie=UTF8&qid=1446823874&sr=8-2&keywords=emetteur+rf+433
il faut ajouter une antenne à cette emetteur RF. Un fil de 17 cms à souder suffit.
https://www.raspberrypi.org/forums/viewtopic.php?f=37&t=66946

/*************************************
      Elaboration scripts cgi:
**************************************/
Les scripts sont mis à disposition dans le répertoire /usr/share/nginx/www/MCZ/cgi-bin
Ces scripts sont propres à mon application, ils sont appelés à chaque commande par ma zibase.

A noter l'utilisation de NGINX comme outil de serveur web.
# apt-get install nginx
# /etc/init.d/nginx start

-> Modifier le fichier /etc/nginx/sites-available/default :

 location /MCZ/cgi-bin/ {
         # Disable gzip (it makes scripts feel slower since they have to complete
         # before getting gzipped)
         gzip off;
         # Fastcgi socket
         fastcgi_pass  unix:/var/run/fcgiwrap.socket;
         # Fastcgi parameters, include the standard ones
         include /etc/nginx/fastcgi_params;
         # Adjust non standard parameters (SCRIPT_FILENAME)
         fastcgi_param SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        }

-> puis relancer NGINX : 
$ nginx -s reload

Exemple de script CGI perl :

  #!/usr/bin/perl -w
       # Tell perl to send a html header.
       # So your browser gets the output
       # rather then <stdout>(command line
       # on the server.)
  use CGI qw(:standard);
  use strict;
  use XML::LibXML;
  
   my $query = new CGI;
   my $valeur_mode = $query->param('Mode'); #donnee d'entree "Mode"
   my $valeur_user = $query->param('User'); #donnee d'entree "User"
   my $valeur_puis = $query->param('Puis'); #donnee d'entree "Puis"
   my $valeur_vent1 = $query->param('Vent1'); #donnee d'entree "Vent1"
   my $valeur_vent2 = $query->param('Vent2'); #donnee d'entree "Vent2"
   my $filename = "parameters.xml";
   my $p = XML::LibXML->new();
   my $d = $p->parse_file($filename);
  
  for my $node ($d->findnodes('//ParametersOfMCZ'))
  {
     my ($modes_node) = $node->findnodes('Modes/text()')
        or next;
     $modes_node->setData($valeur_mode);
     my ($user_node) = $node->findnodes('User/text()')
        or next;
     $user_node->setData($valeur_user);
     my ($puissance_node) = $node->findnodes('Puissance/text()')
        or next;
     $puissance_node->setData($valeur_puis);
     my ($vent1_node) = $node->findnodes('Ventilateur1/text()')
        or next;
     $vent1_node->setData($valeur_vent1);
      my ($vent2_node) = $node->findnodes('Ventilateur2/text()')
        or next;
     $vent2_node->setData($valeur_vent2);
  }
  print $d->toString; #Affiche
  print $d->toFile($filename,0);
  
  print "Content-type: text/html\n\n";
       # print your basic html tags.
       # and the content of them.
  #print "<html><head><title>Hello World!! </title></head>\n";
  #print "<body><h1>Hello world</h1></body></html>\n";


-> Lancer dans un navigateur ou dans un scénario dans la box domotique.
    Exemple pour la valeur du paramètre "Mode": http://<IP>/MCZ/cgi-bin/Mode.cgi?Mode=x
    Exemple pour la valeur du paramètre "Puissance": http://<IP>/MCZ/cgi-bin/Puissance.cgi?Puis=x
    Exemple pour la valeur du paramètre "Ventilateur1": http://<IP>/MCZ/cgi-bin/Ventilateur1.cgi?Vent1=x
    Exemple pour la valeur du paramètre "Ventilateur2": http://<IP>/MCZ/cgi-bin/Ventilateur1.cgi?Vent2=x
    x étant la valeur associée dans la commande.

-> Ne pas oublier de lancer l'application en tâche de fond.
    $ cd /home/pi/Dev/APPLI_MCZ/src
    $ nohup ./APPLI_Emit &
