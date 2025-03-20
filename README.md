# PROJETO-DOCKER---WORDPRESS

Documentação para Instalação do WordPress no Docker com Amazon Linux 

Esta documentação descreve como configurar o WordPress em uma instância EC2 com Amazon Linux utilizando Docker. Além disso, configura-se um Elastic Load Balancer (ELB) da AWS para balancear a carga entre as instâncias EC2 que executam o WordPress, garantindo alta disponibilidade e escalabilidade.

Pré-requisitos
Conta na AWS.
Instância Amazon Linux 2 EC2 criada.
Acesso SSH à instância.
Docker instalado na instância EC2.
Elastic Load Balancer (ELB) configurado para distribuir o tráfego.
Banco de dados MySQL configurado, pode ser usado Amazon RDS ou MySQL local.
Domínio configurado (opcional) para associar ao Load Balancer.
Passo 1: Preparar a Instância EC2 com Amazon Linux
Conectar à Instância EC2 via SSH:

Abra o terminal no seu computador.
Conecte-se à sua instância EC2 usando o comando SSH:
bash
Copiar
ssh -i /caminho/para/sua-chave.pem ec2-user@<IP-da-sua-instância>
Atualizar o Sistema:

Execute o seguinte comando para garantir que todos os pacotes estejam atualizados:
bash
Copiar
sudo yum update -y
Instalar o Docker:

Instale o Docker na instância EC2 usando os seguintes comandos:
bash
Copiar
sudo amazon-linux-extras enable docker
sudo yum install docker -y
sudo service docker start
sudo usermod -aG docker ec2-user
Verificar o Status do Docker:

Verifique se o Docker está em funcionamento com o comando:
bash
Copiar
sudo systemctl status docker
Reiniciar a Instância EC2 (se necessário):

Caso a adição do usuário ao grupo Docker não tenha sido efetivada, faça logout e login novamente ou reinicie a instância EC2.
Passo 2: Configurar o Banco de Dados MySQL
Neste exemplo, vamos usar o Amazon RDS para o banco de dados do WordPress.

Criar uma Instância RDS com MySQL:

Acesse o AWS Management Console.
Vá para o serviço RDS e crie uma instância do banco de dados MySQL.
Defina as configurações do banco, como nome do banco de dados, usuário e senha.
Certifique-se de configurar o Grupo de Segurança para permitir conexões da sua instância EC2.
Obter o Endpoint do Banco de Dados:

Após a criação da instância RDS, copie o endpoint da instância para usar na configuração do WordPress.
Passo 3: Criar e Configurar o Docker para WordPress
Baixar e Executar o Docker Compose:

Vamos usar o Docker Compose para orquestrar o WordPress e o banco de dados MySQL em containers separados.
Baixe o Docker Compose:
bash
Copiar
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
Criar o arquivo docker-compose.yml:

Na sua instância EC2, crie um diretório para o projeto WordPress e acesse-o:
bash
Copiar
mkdir ~/wordpress-docker
cd ~/wordpress-docker
Crie um arquivo chamado docker-compose.yml e adicione o seguinte conteúdo:
yaml
Copiar
version: '3.1'

services:
  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: <RDS-ENDPOINT>:3306
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: <DB-USER>
      WORDPRESS_DB_PASSWORD: <DB-PASSWORD>
    networks:
      - wordpress-network
  
  mysql:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: <DB-PASSWORD>
      MYSQL_DATABASE: wordpress
    networks:
      - wordpress-network

networks:
  wordpress-network:
    driver: bridge
Substitua os seguintes valores:

<RDS-ENDPOINT>: O endpoint do banco de dados MySQL no Amazon RDS.
<DB-USER> e <DB-PASSWORD>: O nome de usuário e senha do banco de dados MySQL.
Iniciar os Containers:

Execute o comando abaixo para iniciar o WordPress e o MySQL no Docker:

bash
Copiar
sudo docker-compose up -d
Verifique se os containers estão funcionando:

bash
Copiar
sudo docker ps
Passo 4: Configurar o Elastic Load Balancer (ELB)
Criar um Elastic Load Balancer:

Acesse o Console da AWS e vá para o serviço EC2.
No menu à esquerda, clique em Load Balancers e em seguida, clique em Create Load Balancer.
Selecione Application Load Balancer (ALB).
Configure as opções de rede e selecione o VPC onde as instâncias EC2 estão localizadas.
Configurar o Listener:

Para a comunicação HTTP, o Listener deve estar configurado na porta 80. Se quiser HTTPS, adicione outro Listener na porta 443 e associe um certificado SSL.
Registrar Instâncias no Target Group:

No ALB, crie um Target Group para registrar as instâncias EC2.
Registre as instâncias EC2 com o WordPress em execução no Target Group.
Configure a verificação de saúde para garantir que o tráfego seja direcionado apenas para instâncias saudáveis.
Testar o Load Balancer:

Após configurar o ALB, você terá um DNS público associado ao Load Balancer. Acesse o DNS e verifique se o WordPress está sendo servido corretamente.
Passo 5: Finalizando a Instalação e Testes
Acessar o WordPress:

Após configurar o Load Balancer, use o DNS fornecido pelo ALB para acessar o WordPress através de um navegador.
Finalize a instalação do WordPress, configurando o site, o tema e os plugins.
Escalabilidade:

Adicione mais instâncias EC2 ao grupo de destino do ALB, conforme necessário, para aumentar a capacidade e garantir alta disponibilidade.
Considerações Finais
Segurança: Certifique-se de configurar segurança de rede adequadamente, usando Security Groups para proteger o acesso às instâncias EC2 e ao banco de dados.
Escalabilidade: A configuração com Elastic Load Balancer (ELB) permite escalar o WordPress facilmente, distribuindo o tráfego entre múltiplas instâncias EC2.
Backup: Não se esqueça de configurar backups automáticos para os dados do WordPress e para o banco de dados no Amazon RDS.
