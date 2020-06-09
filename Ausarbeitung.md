# Ausarbeitung OS-Management
***
## **Ablauf der Installation der VM**
***
Ausgewähltes Linux Distro: Ubunto Server 20.04.
1.	Tastatur konfigurieren  
1.1.	Layout auswählen: Deutsch  
1.2.	Variante auswählen: Deutsch
2.	Netzwerkverbindungen einrichten (Auswahl des Network-Interfaces)  
2.1	Default gelassen
3. Konfiguration eines Proxys  
3.1	Default gelassen  
4.	Spiegelserver verwenden    
4.1	Default  
5.	„Guided storage layout“ konfigurieren  
5.1.	Default  
6.	Speicher konfigurieren   
6.1	Default  
7.	Benutzer setup  
7.1.	Eingabe name  
7.2.	Server name  
7.3.	Username  
7.4.	Password  
8.	SSH Setup  
8.1.	Installation des SSH-Daemon ,OpenSSH   
9.	Featured Server Snaps   
9.1.	Default  
10.	Installation   

Nach der Installation und den Neustart des OS starten wir mit der Konfiguration des SSH-Daemons (OpenSSH).  

***
## **Konfiguration von OpenSSH:**
***
1.	Öffnen des config  files:  
1.1	Command: $ sudo vim /etc/ssh/shh_config  
2.	Ändern des Ports für die SSH-Verbindung (Default 22)  
2.1.	Feld für den SSH-Port auskommentieren und Port angeben  
2.1.1.	In meinen Fall: Port 65100
3.	Verbindung von non root User autorisieren
3.1.	Hierzu müssen sie wieder das Feld PermitRootLogin auskommentieren  
3.1.1.	 Dann müssen sie nur noch dem Feld PermitRootLogin „no“ anfügen  
4.	Nun geht es darum in ihren Netzwerkeinstellungen für die VM das Port Forwarding zu aktivieren  
4.1.	Bei VirtualBox sehen sie einen Tab Maschine. Von hier aus müssen sie auf Ändern->Netzwerk->Erweitert->Port-Weiterleitung gehen  
4.2.	Meine Einstellungen waren :  Host-IP:  127.0.0.1| Host-Port: 44444 | Gast-IP 10.0.2.15 | Gast-Port: 65100  
4.3.	Die Host-IP wäre in meinen Fall die Loopback Adresse     
4.4.	Der Host-Port kann frei gewählt werden  
4.5.	Die lokale IP-Adresse von der VM wird bei der Gast-IP eingetragen   
4.5.1.	Falls sie diese nicht wissen geben sie das Command: $ Ip addr auf Ihrer VM ein   
4.5.1.1. In meinen Fall ist die IP: 10.0.2.15  
4.6.	Nun bei dem Gast-Port ist es wichtig, dass sie den Port eingeben über den ihre SSH-Service erreichbar ist. Dies haben vorhin in der config Datei für den SSH-Service definiert (in meinem Fall war dies 65100)
4.6.1.	Host-IP = Loopback (127.0.0.1)
4.6.2.	Host-Port= Beispielweise 44444  
4.6.3.	Lokale IP von VM: 10.0.2.15  
4.6.4.	Der im config file definierte Port: 65100  

Nun ist es Ihnen möglich sich zu ihrem Server mittels SSH von ihrer Hostmaschine zu verbinden.  

***
## **Verbindung zu der VM herstellen mittels SSH**
***
Dazu müssen folgende Schritte beachtet werden:
1.	Zuerst einmal muss auf ihrer Hostmaschine ein SSH Public Key file generiert werden    
1.1	Das Command hierfür: $ shh-keygen -t rsa  
1.2	Anschließend werden sie nach dem Speicherort gefragt  
1.2.1	Mit „Enter“ speichern sie den Key Default-Ordner (/root/.ssh/id_rsa)  
1.3	Ebenso können sie Passphrase definieren  
1.3.1	Falls sie keine haben wollen einfach leer lassen und mit „Enter“ bestätigen  
2.	Nun müssen sie den Public Key von der Hostmaschine an die VM schicken  
2.1	Das Command hierfür: $ scp -P 44444 ~/.ssh/id_rsa.pub  tom@127.0.0.1:/home/tom  
2.1.1	Bei -P: der Host-Port der in den Netzwerkeinstellungen der VM für das Port-Forwarding definiert wurde (in meinem Fall 44444)  
2.1.2	Dann kommt das Verzeichnis in welcher der SSH Public Key gespeichert wurde  
2.1.3 Am Schluss führen sie ihren Username an begleitet von ihrer Host-IP Adresse und das Verzeichnis, in dem der Key auf der VM gespeichert werden soll   
3.	Nun müssen sie das Public Key File auf ihrer VM zu den .ssh/authorized _keys hinzufügen  
3.1	Hierzu geben sie das Command: cat id_rsa.pub  >> .ssh/authorized_keys  
4.	Nun gehen sie wieder auf ihr SSH Config file, öffnen es $ sudo vim /etc/ssh/sshd_config und nehmen folgende Änderungen vor:  
4.1	Kommentieren sie „PasswordAuthentication“ aus und ergänzen mit:  PasswordAuthentication no  
4.2	Kommentieren sie „PubKeyAuthentication“ aus und ergänzen mit: PubKeyAuthentication yes  
5. Nun können sie vom Host aus mittels SSH auf Server verbinden: 
5.1	Command: $ ssh -p 44444 tom@127.0.0.1

***
## **Firewall konfigurieren**
***
1.	Zuerst alle Verbindungen zu den Ports unterbinden  
1.1	Command: $ ufw default deny  
2.	Nun müssen sie den Port für die einkommenden SSH Verbindungen öffnen  
2.1	Hierzu müssen Verbindungen über den Port, auf den ihr SSH-Service läuft, zugelassen werden  
2.1.1	Command: $ sudo allow 65100 (Port von SSH auf VM)  
3.	Um die Firewall dann zu aktivieren  
3.1	Command: $ sudo ufw enable  

