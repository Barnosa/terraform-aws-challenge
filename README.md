
Este repositório contém código Terraform para provisionar uma infraestrutura básica na AWS, distribuída em duas tarefas principais. O foco é o provisionamento de uma VPC, instância EC2, chaves de acesso SSH, e outros componentes essenciais para o funcionamento de uma aplicação em ambiente de desenvolvimento.

Tarefa 1: Provisionamento de Infraestrutura AWS
Visão Geral
Esta tarefa provisiona uma infraestrutura na AWS com componentes essenciais para o funcionamento de uma aplicação. O código cria recursos como VPC, sub-rede, grupo de segurança, e uma instância EC2 utilizando o Debian 12.

Recursos Provisionados
🔐 Chave Privada
Recurso: tls_private_key.ec2_key
Descrição: Gera uma chave privada RSA (2048 bits) para acessar a instância EC2 via SSH.
🔑 Par de Chaves
Recurso: aws_key_pair.ec2_key_pair
Descrição: Cria um par de chaves AWS utilizando a chave pública gerada anteriormente.
🌐 VPC (Virtual Private Cloud)
Recurso: aws_vpc.main_vpc
Descrição: Cria uma VPC com o bloco CIDR 10.0.0.0/16, com suporte a DNS habilitado.
🛰️ Sub-rede
Recurso: aws_subnet.main_subnet
Descrição: Cria uma sub-rede na VPC com o bloco CIDR 10.0.1.0/24, localizada na zona de disponibilidade us-east-1a.
🌍 Internet Gateway
Recurso: aws_internet_gateway.main_igw
Descrição: Cria um gateway de internet para permitir tráfego externo na VPC.
🛣️ Tabela de Rotas
Recurso: aws_route_table.main_route_table
Descrição: Cria uma tabela de rotas para direcionar o tráfego de saída para o gateway de internet.
🔗 Associação da Tabela de Rotas
Recurso: aws_route_table_association.main_association
Descrição: Associa a sub-rede à tabela de rotas, conectando-a ao gateway de internet.
🔒 Grupo de Segurança
Recurso: aws_security_group.main_sg
Descrição: Cria um grupo de segurança que permite:
Acesso SSH (porta 22) de qualquer IP.
Todo tráfego de saída.
🖼️ AMI (Amazon Machine Image)
Recurso: data.aws_ami.debian12
Descrição: Seleciona automaticamente a AMI mais recente do Debian 12 (64 bits).
💻 Instância EC2
Recurso: aws_instance.debian_ec2
Descrição: Provisiona uma instância EC2 do tipo t2.micro, utilizando a AMI do Debian 12, configurada para acesso via SSH com as chaves geradas. Um script de inicialização atualiza o sistema operacional automaticamente.
Saídas
🔑 Chave Privada: A chave privada gerada para acessar a instância EC2 (sensível).
🌐 IP Público da EC2: O endereço IP público atribuído à instância EC2.
Tarefa 2: Provisionamento de Instância EC2 com Terraform
Visão Geral
A Tarefa 2 tem como objetivo provisionar uma infraestrutura básica na AWS, utilizando o Terraform para automatizar o processo. Este código cria uma VPC, uma instância EC2 com Debian 12, chave SSH para acesso seguro, e outros recursos necessários para conectar a instância à internet.

Estrutura do Código
📦 Configuração do Terraform
hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
  required_version = ">= 1.0"
}
Versão do Provedor AWS: Define a versão do provedor AWS a ser utilizada (~> 4.0).
Versão do Terraform: Garante que a versão do Terraform seja pelo menos 1.0.
🌍 Provedor AWS
hcl
provider "aws" {
  region = "us-east-1"
}
Define a região da AWS onde os recursos serão provisionados, no caso, a us-east-1.
🛠️ Variáveis e Prefixo
hcl
Copiar código
variable "projeto" {
  description = "Nome do projeto"
  type        = string
  default     = "VExpenses"
}

variable "candidato" {
  description = "Nome do candidato"
  type        = string
  default     = "SeuNome"
}

locals {
  name_prefix = "${var.projeto}-${var.candidato}"
}
Variáveis de Entrada: Nome do projeto e do candidato, que são usados para criar um prefixo de nome para identificar os recursos.
🔑 Chave SSH
hcl
resource "tls_private_key" "ec2_key" {
  algorithm = "RSA"
  rsa_bits  = 2048
}

resource "aws_key_pair" "ec2_key_pair" {
  key_name   = "${local.name_prefix}-key"
  public_key = tls_private_key.ec2_key.public_key_openssh
}
Geração de Chave Privada: Gera uma chave privada de 2048 bits para acessar a instância EC2 via SSH.
Par de Chaves AWS: Cria um par de chaves na AWS usando a chave pública gerada, para associá-la à instância EC2.
🌐 VPC e Sub-rede
hcl
Copiar código
resource "aws_vpc" "main_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = {
    Name        = "${local.name_prefix}-vpc"
    Environment = "Development"
  }
}

resource "aws_subnet" "main_subnet" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  tags = {
    Name        = "${local.name_prefix}-subnet"
    Environment = "Development"
  }
}
VPC (Virtual Private Cloud): Cria uma VPC com o bloco CIDR 10.0.0.0/16, com suporte a DNS.
Sub-rede: Cria uma sub-rede dentro da VPC, na zona de disponibilidade us-east-1a.
🛰️ Internet Gateway e Rotas
hcl
Copiar código
resource "aws_internet_gateway" "main_igw" {
  vpc_id = aws_vpc.main_vpc.id
  tags = {
    Name        = "${local.name_prefix}-igw"
    Environment = "Development"
  }
}

resource "aws_route_table" "main_route_table" {
  vpc_id = aws_vpc.main_vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main_igw.id
  }
  tags = {
    Name        = "${local.name_prefix}-route_table"
    Environment = "Development"
  }
}

resource "aws_route_table_association" "main_association" {
  subnet_id      = aws_subnet.main_subnet.id
  route_table_id = aws_route_table.main_route_table.id
}
Internet Gateway: Permite que a VPC se conecte à internet.
Tabela de Rotas: Define a rota padrão (0.0.0.0/0) para o tráfego de saída da VPC, utilizando o Internet Gateway.
Associação da Tabela de Rotas: Associa a sub-rede à tabela de rotas para garantir que o tráfego tenha acesso à internet.
🔒 Grupo de Segurança
hcl
resource "aws_security_group" "main_sg" {
  name        = "${local.name_prefix}-sg"
  description = "Permitir SSH de qualquer lugar e todo o tráfego de saída"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    description      = "Permitir SSH de qualquer lugar"
    from_port        = 22
    to_port          = 22
    protocol         = "tcp"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  egress {
    description      = "Permitir todo tráfego de saída"
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
}
Ingress Rule: Permite o acesso SSH (porta 22) de qualquer IP (IPv4 e IPv6).
Egress Rule: Permite todo o tráfego de saída para a internet.
💻 Instância EC2
hcl
data "aws_ami" "debian12" {
  most_recent = true
  filter {
    name   = "debian-12-amd64-*"
    values = ["debian-12-amd64-*"]
  }
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  owners = ["679593333241"]
}

resource "aws_instance" "debian_ec2" {
  ami             = data.aws_ami.debian12.id
  instance_type   = "t2.micro"
  subnet_id       = aws_subnet.main_subnet.id
  key_name        = aws_key_pair.ec2_key_pair.key_name
  security_groups = [aws_security_group.main_sg.name]
  associate_public_ip_address = true

  root_block_device {
    volume_size           = 20
    volume_type           = "gp2"
    delete_on_termination = true
  }

  user_data = <<-EOF
              #!/bin/bash
              apt-get update -y
              apt-get upgrade -y
              EOF

  tags = {
    Name        = "${local.name_prefix}-ec2"
    Environment = "Development"
  }
}
AMI (Amazon Machine Image): Seleciona a AMI mais recente do Debian 12 para criar a instância.
Instância EC2: Cria uma instância EC2 do tipo t2.micro, com um script de inicialização que atualiza o sistema operacional automaticamente. O volume de armazenamento é configurado com 20 GB.
📤 Saídas
hcl
output "private_key" {
  description = "Chave privada para acessar a instância EC2"
  value       = tls_private_key.ec2_key.private_key_pem
  sensitive   = true
}

output "ec2_public_ip" {
  description = "Endereço IP público da instância EC2"
  value       = aws_instance.debian_ec2.public_ip
}
Chave Privada: Exibe a chave privada para acessar a instância EC2 (sensível).
IP Público da EC2: Exibe o endereço IP público da instância EC2, utilizado para se conectar via SSH.
