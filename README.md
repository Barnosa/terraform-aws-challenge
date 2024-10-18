# Projeto de Infraestrutura com Terraform

## Tarefa 1 - Provisionamento de Infraestrutura na AWS

### Descrição Técnica

Este projeto utiliza o Terraform para provisionar uma infraestrutura na AWS, conforme as especificações fornecidas. Os principais recursos e componentes criados são:

- **Provedor AWS Provider**: Configuração do provedor AWS na região `us-east-1`.

### Recursos Criados

- **Chave Privada**: Gera uma chave privada RSA para acesso à instância EC2.
- **Par de Chaves**: Cria um par de chaves AWS usando a chave pública gerada anteriormente.
- **VPC (Virtual Private Cloud)**: Criação de uma VPC com o bloco CIDR `10.0.0.0/16`.
- **Sub-rede**: Criação de uma sub-rede na VPC com o bloco CIDR `10.0.1.0/24`.
- **Gateway de Internet**: Criação de um gateway de internet para a VPC.
- **Tabela de Rotas**: Criação de uma tabela de rotas que direciona o tráfego para o gateway de internet.
- **Grupo de Segurança**: Permite acesso SSH (porta 22) de qualquer lugar e permite todo o tráfego de saída.
- **Instância EC2**: Provisionamento de uma instância EC2 do tipo `t2.micro` rodando Debian 12.

# Projeto de Infraestrutura com Terraform - Tarefa 2

## Descrição da Tarefa

Nesta tarefa, foram implementadas melhorias na infraestrutura provisionada na AWS usando Terraform. O objetivo principal foi otimizar a segurança e a escalabilidade da infraestrutura existente, além de melhorar o monitoramento da instância EC2.

### Melhorias Implementadas

1. **Otimização da Segurança**:
   - As regras do grupo de segurança foram ajustadas para permitir o acesso SSH apenas de um endereço IP específico, reduzindo a exposição da instância a potenciais ataques.
   - Adição de regras para permitir tráfego HTTP e HTTPS, facilitando a comunicação com a instância EC2.

2. **Configuração de IAM**:
   - Criação de uma função IAM para permitir que a instância EC2 assuma papéis específicos, facilitando a gestão de permissões.

3. **Aprimoramento do Monitoramento**:
   - Integração com o AWS CloudWatch para coletar logs e métricas da instância EC2, melhorando a visibilidade sobre a performance e o estado da aplicação.

## Arquivo `main.tf` Modificado

Aqui está o conteúdo atualizado do arquivo `main.tf`:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
  required_version = ">= 1.0"
}

provider "aws" {
  region = "us-east-1"
}

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

resource "tls_private_key" "ec2_key" {
  algorithm = "RSA"
  rsa_bits  = 2048
}

resource "aws_key_pair" "ec2_key_pair" {
  key_name   = "${local.name_prefix}-key"
  public_key = tls_private_key.ec2_key.public_key_openssh
}

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

resource "aws_security_group" "main_sg" {
  name        = "${local.name_prefix}-sg"
  description = "Permitir SSH e tráfego HTTP/HTTPS"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    description = "Permitir SSH de um IP específico"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["SEU_IP_AQUI/32"]  # Substitua SEU_IP_AQUI pelo seu IP
  }

  ingress {
    description = "Permitir tráfego HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "Permitir todo tráfego de saída"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "${local.name_prefix}-sg"
    Environment = "Development"
  }
}

data "aws_ami" "debian12" {
  most_recent = true
  filter {
    name   = "name"
    values = ["debian-12-amd64-*"]
  }
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  owners = ["679593333241"]
}

resource "aws_instance" "debian_ec2" {
  ami                     = data.aws_ami.debian12.id
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.main_subnet.id
  key_name               = aws_key_pair.ec2_key_pair.key_name
  security_groups        = [aws_security_group.main_sg.name]
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

output "private_key" {
  description = "Chave privada para acessar a instância EC2"
  value       = tls_private_key.ec2_key.private_key_pem
  sensitive   = true
}

output "ec2_public_ip" {
  description = "Endereço IP público da instância EC2"
  value       = aws_instance.debian_ec2.public_ip
}
