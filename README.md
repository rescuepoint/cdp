# Processo de instalação do Cloudera CDP de teste
Esse material tem o intuito de auxiliar no processo de instalação do Cloudera CDP fornecendo um material passo a passo desde o desenho de arquitetura até a automação do processo em scripts Ansible.

## Desenho de arquitetura
Abaixo podemos ver o desenho de arquitetura sugerido para o projeto, separando redes privadas e publicas, usando um bastion para acessar os servers do Cloudera e um Loadbalance que vai ficar na ponta apontando somente para os serviços de acordo com os listeners criados no Load Balance. Segue abaixo o desenho:
