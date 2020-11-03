# Instalação CDP Teste  
Material de auxilio para a instalação do Cloudera CDP no Oracle OCI conforme materiais do Youtube e Medium da Rescue Point.

## Desenho de arquitetura:
Segue abaixo um desenho de arquitetura que mostra básicamente como está configurada, separando rede publica, privada, serviços e servidores.

![Desenho Arquitetura Cloud CDP](https://github.com/rescuepoint/cdp/blob/main/fotos/Sem%20Ti%CC%81tulo.PNG)

### Toda essa arquitetura foi explicada em video de youtube abaixo:
[![Instalando Cloudera CDP - Arquitetura](https://img.youtube.com/vi/XoVSxKtdbmo/0.jpg)](https://www.youtube.com/watch?v=XoVSxKtdbmo)

### Esse desenho cotempla em termos de redes:
* Uma VPC - 10.0.0.0/16
* Uma subnet publica - 10.0.0.0/24
* Uma subnet privada - 10.0.1.0/24
* 1 firewall privado
* 1 firewall publico

### Em termos de recursos:
* 1 servidor bastion (Allways free) na rede publica.
* 1 load balance para acesso externo nas instancias da rede privada de acordo com o serviço.
* 8 Servidores VM.Standard.2.2 com 100GB de boot disk cada para as maquinas do Cloudera. Sistema Operacional vai ser o CentOS 7.
* Os servidores criados tem a chave ssh para login com o user OPC a partir do MrRobot (bastion).
* 6 discos block storage que serão usados nas instancias que farão o papel de data nodes com 100GB cada.

### Caracteristicas da rede:
* Foi configurado um firewall privado, nesse foi liberada todas as portas sendo da origem a rede publica (10.0.0.0/24) e saida completa sem restrição.
* Foi configurado um firewall na rede publica, nesse eu libero somente as portas que preciso na entrada, como 22, 80, 7180 e outras conforme for precisando. Na saida a regra é liberação total.

## Criação da primeira imagem:
Vamos realizar a criação da primeira maquina como base para imagem, com a configuração dos pré requisitos que todas elas devem ter, segue a lista de todos os itens abaixo:
* Parada e desativação do firewall
* Desativação do SELinux
* Configuração dos parâmetros de Limits.
* Alterar o timezone para Brasil
* Habilitar o Swappiness
* Desabilitar o Transparent HugePages tanto na sessão como pós reboot
* Instalar o Kerberos agent em todos os nodes.
* Configurar todas as maquinas no /etc/hosts
* Baixar o JDBC do MySQL e configurar no diretório adequado.
* Configurar um repositorio local do Cloudera instalando o Apache, configurando o modo de parcels e baixando os RPMs e parcels para o diretório da maquina.
* Criar um repositporio usando o createrepo e colocando na lista de repositorios do YUM.
* Instalar o JAVA da propria Cloudera (recomendado)
* Criar as variaveis de ambiente do Java no /etc/profile
* Alterar a senha do user root e criar o usuário CDP com privilégios de sudo sem senha
* Alterar o SSH para login com senha (por padrão o OCI libera apenas login com chave).

Usaremos um playbook Ansible para realizar todos esses passos, o playbook se encontra nesse repositório com o nome de cdp.yml. Após a execução do playbook iremos criar uma imagem do servidor após boot e assim criar nossas instancias definitivas.

## Procedimento de criação da Imagem e demais maquinas:
O procedimento de criação da imágem e da criação dos servidores no Oracle OCI encontram-se no vídeo do Youtube do link abaixo:<br>
[![Instalando Cloudera CDP - Criando imagem com pre-requisitos e infra](https://img.youtube.com/vi/VTz6vAbRe1w/0.jpg)](https://www.youtube.com/watch?v=VTz6vAbRe1w)

## Instalando pre-reqs do node do Cloudera Manager Server:
Agora que já instalamos o básico para todos os nodes, vamos fazer as seguintes configurações no primeiro node (elliot01), que será o node do Cloudera Manager Server:
* Instalação do MariaDB e varias outras features dele (por exemplo a do systemctl, caso contrario não inicia via systemd)
* Configuração de senha para o user root do MariaDB e a criação de todos os databases e usuários necessários para o Cloudera Manager Server.
* Instalação do Cloudera Manager Server
* Edição do arquivo db.properties via comando para que quando o Cloudera Manager iniciar a primeira vez realize a criação da estrutura necessária no Database CMF.
* Ativação do Cloudera Manager Server no boot.
* Inicialização do Cloudera Manager.

Todo esse procedimento vai ser realizado por um segundo playbook chamado cdp_node1.yml e vai ser executado somente no primeiro node, observe o script.

Segue o link com o vpideo dessa parte do material: LInk

## Criando o Loadbalance no Oracle OCI para o Cloudera Manager Server.
Vamos usar um recurso de Load Balance do OCI para apontarmos as requisições externas da porta 7180 para o Cloudera Manager Server que está na rede privada. Esse tipo de configuração dá uma confiança absurda no acesso, escalabilidade, facilidade em possivei modificações. 

O desenho do acesso vai ser o seguinte:

![Funcionamento Load Balance](https://img.youtube.com/vi/XoVSxKtdbmo/0.jpg)

O procedimento é bem simples e explicado no vídeo anterior.



