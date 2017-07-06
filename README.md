# Bind-DNS-PRTG
Captura variáveis do arquivo de stats do bind e gerar arquivo de configuração para ser capturado e gerado gráfico para o PRTG.

1. Editar o arquivo no servidor de DNS:
    
        root@dns:/# nano /etc/bind/named.conf.options
incluir: 

        zone-statistics yes;
        statistics-file "/var/log/named/named.log";

2. ir até /var/log/named, executar o comando:

        root@dns:/var/log/named# rndc stats


verificar se o arquivo ficou assim: 

    +++ Statistics Dump +++ (1499366581)
    ++ Incoming Requests ++
                40386068 QUERY
                       1 STATUS
                      98 UPDATE
    ++ Incoming Queries ++
                33939822 A
                   50216 NS
                    2649 CNAME
                    8960 SOA
                 1182413 PTR
                  524090 MX
                  136993 TXT
                 4455733 AAAA
                   67314 SRV
                      30 NAPTR
                       6 A6
                      45 DS
                       9 DNSKEY
                   12111 SPF
                    4964 ANY
    ++ Outgoing Queries ++
    [View: default]
             7152922 A
               96248 NS
                 472 CNAME
                1162 SOA
               62979 PTR
              727373 MX
               67316 TXT
             1581883 AAAA
                3417 SRV
                  19 NAPTR
             1019773 DS
               36176 DNSKEY
                 341 ANY
3. programar o crontab para apagar e gerar o arquivo de estatisticas a cada 5min. 
        
        root@dns:/# crontab -e

colocar o comando abaixo. 
        
        */5 * * * * rm -rf /var/log/named/named.log && /usr/sbin/rndc stats

4. Instalar um servidor Debian para capturar os dados do bind. Não usar o mesmo servidor. 
no debian: 

       root@debian:/# apt-get install apache2 lftp

editar a porta do apache para 8080, ou como preferir, mudar a linha Listen 80 para 8080 

        root@debian:/# nano /etc/apache2/ports.conf

5.Criar o Script abaixo:  

        root@debian:/# nano /var/www/html/vai.sh

colocar o script: 

    #!/bin/bash
    #script para caputar dos dados de Stats do Bind e criar o arquivo para o PRTG.
    #criado por Osni Silva.

    #faz download do arquivo named.log do meu servidor bind.
    lftp -e "mirror /var/log/named /var/www/html;quit" -p 2222 -u login,senha sftp://ipdoservidordedns-bind

    #limpa o arquivo /var/www/html/prtg.txt
    rm -rf /var/www/html/prtg3.txt

    #pega o nome dos arquivos e gera os canais
    echo ; grep "Cache DB" /var/www/html/named.log -A 16 | grep "A" | awk '{print "[" $1 "]" }' | sed -n '1p' >> /var/www/html/prtg3.txt
    echo ; grep "Cache DB" /var/www/html/named.log -A 16 | grep "NS" | awk '{print "[" $1 "]" }' | sed -n '1p' >> /var/www/html/prtg3.txt
    echo ; grep "Cache DB" /var/www/html/named.log -A 16 | grep "CNAME" | awk '{print "[" $1 "]" }' | sed -n '1p' >> /var/www/html/prtg3.txt
    echo ; grep "Cache DB" /var/www/html/named.log -A 16 | grep "SOA" | awk '{print "[" $1 "]" }' | sed -n '1p' >> /var/www/html/prtg3.txt
    echo ; grep "Cache DB" /var/www/html/named.log -A 16 | grep "PTR" | awk '{print "[" $1 "]" }' | sed -n '1p' >> /var/www/html/prtg3.txt
    echo ; grep "Cache DB" /var/www/html/named.log -A 16 | grep "HINFO" | awk '{print "[" $1 "]" }' | sed -n '1p' >> /var/www/html/prtg3.txt
    echo ; grep "Cache DB" /var/www/html/named.log -A 16 | grep "MX" | awk '{print "[" $1 "]" }' | sed -n '1p' >> /var/www/html/prtg3.txt
    echo ; grep "Cache DB" /var/www/html/named.log -A 16 | grep "TXT" | awk '{print "[" $1 "]" }' | sed -n '1p' >> /var/www/html/prtg3.txt
    echo ; grep "Cache DB" /var/www/html/named.log -A 16 | grep "AAAA" | awk '{print "[" $1 "]" }' | sed -n '1p' >> /var/www/html/prtg3.txt
    echo ; grep "Cache DB" /var/www/html/named.log -A 16 | grep "SRV" | awk '{print "[" $1 "]" }' | sed -n '1p' >> /var/www/html/prtg3.txt

Dar permisão 

    root@debian:/# chmod +x /var/www/html/vai.sh

6. Programar no crontab para rodar o scrip vai.sh de 5 em 5 minutos. 

        */1 * * * * sh /var/www/html/./vai.sh

7. No diretorio do /var/www/html deve conter os arquivos: 

        root@debian:/# ls -ln /var/www/html
        total 40
        -rw-r--r-- 1 0 0 14124 Jul  6  2017 named.log
        -rw-r--r-- 1 0 0    64 Jul  6 14:20 prtg.txt
        -rwxr-xr-x 1 0 0  1614 Jul  6 09:58 vai3.sh
        
        obs.: named.log é o arquivo que o script baixou do servidor de dns bind.
              prtg.txt é o arquivo formatado para o prtg gerar os canais.
              vai3.sh é nosso script. 
  
Até  mais...


