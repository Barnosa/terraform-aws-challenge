 ##Tarefa -1
Este projeto utiliza o Terraform para provisionar uma infraestrutura na AWS, conforme o código fornecido pela empresa. A seguir, estão os principais recursos e componentes criados pelo código:
Descrição Técnica 
Provedor AWS
Provider: Configura o provedor AWS na região us-east-1.
Variáveis
projeto: Nome do projeto (default: VExpenses).
candidato: Nome do candidato (default: SeuNome).
Recursos Criados
Chave Privada
tls_private_key.ec2_key: Gera uma chave privada RSA para acesso à instância EC2.
Key Pair
aws_key_pair.ec2_key_pair: Cria um par de chaves AWS utilizando a chave pública gerada anteriormente.
VPC (Virtual Private Cloud)
aws_vpc.main_vpc: Cria uma VPC com o bloco CIDR 10.0.0.0/16, com suporte a DNS habilitado.
Subnet
aws_subnet.main_subnet: Cria uma sub-rede na VPC com o bloco CIDR 10.0.1.0/24, na zona de disponibilidade us-east-1a.
Internet Gateway
aws_internet_gateway.main_igw: Cria um gateway de internet para a VPC.
Tabela de Rotas
aws_route_table.main_route_table: Cria uma tabela de rotas que direciona o tráfego para o gateway de internet.
Associação da Tabela de Rotas
aws_route_table_association.main_association: Associa a sub-rede à tabela de rotas.
Grupo de Segurança
aws_security_group.main_sg: Cria um grupo de segurança que permite acesso SSH (porta 22) de qualquer lugar e permite todo o tráfego de saída.
AMI (Amazon Machine Image)
data.aws_ami.debian12: Seleciona a AMI mais recente do Debian 12.
Instância EC2
aws_instance.debian_ec2: Provisiona uma instância EC2 do tipo t2.micro na sub-rede criada, com o AMI Debian 12 e configuração para acesso via chave SSH. O script de inicialização atualiza o sistema.
Saídas
Chave Privada: A chave privada gerada para acesso à instância EC2 (sensível).
IP Público da EC2: O endereço IP público atribuído à instância EC2.



Tarefa 2: Provisionamento de Instância EC2 com Terraform
Descrição
Esta tarefa utiliza o Terraform para provisionar uma infraestrutura básica na AWS que inclui:

Uma VPC configurada com uma sub-rede.
Um Internet Gateway para permitir o tráfego da internet.
Uma instância EC2 rodando Debian 12 com acesso SSH habilitado.
Um par de chaves SSH gerado para a conexão com a instância.
Um grupo de segurança que permite acesso SSH de qualquer lugar e libera todo o tráfego de saída.
