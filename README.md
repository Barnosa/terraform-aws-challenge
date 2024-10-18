# Projeto Terraform - Infraestrutura na AWS

## 1. Análise Técnica do Código Terraform

Este projeto utiliza Terraform para provisionar uma infraestrutura básica na AWS, que inclui os seguintes componentes:

1. **Provedor AWS**: Define a região `us-east-1` onde os recursos serão criados.
  
2. **Variáveis**:
   - `projeto`: Nome do projeto.
   - `candidato`: Nome do candidato.

3. **Chave Privada TLS**: 
   - Um par de chaves RSA é gerado para acesso SSH à instância EC2.

4. **VPC**: 
   - Criação de uma VPC com CIDR `10.0.0.0/16`.

5. **Sub-rede**: 
   - Criação de uma sub-rede com CIDR `10.0.1.0/24`.

6. **Gateway de Internet**: 
   - Um gateway é criado para permitir acesso à internet.

7. **Tabela de Roteamento**: 
   - A tabela de roteamento é configurada para direcionar o tráfego da sub-rede para o gateway de internet.

8. **Grupo de Segurança**: 
   - Grupo que permite acesso SSH a partir de um IP específico e permite todo o tráfego de saída.

9. **AMI**: 
   - Busca pela imagem mais recente do Debian 12 para a instância EC2.

10. **Instância EC2**: 
    - Criação de uma instância EC2 usando a AMI do Debian 12 e tipo `t2.micro`.
    - Configuração para ter um IP público e associar-se ao grupo de segurança.
    - O Nginx é instalado automaticamente durante a inicialização.

## 2. Modificação e Melhoria do Código Terraform

### Melhorias de Segurança

- O código foi modificado para restringir o acesso SSH a um IP específico, aumentando a segurança do ambiente.

### Automação da Instalação do Nginx

- A instância EC2 é configurada para instalar e iniciar o servidor Nginx automaticamente utilizando o seguinte script `user_data`:

```bash
#!/bin/bash
apt-get update
apt-get install -y nginx
systemctl start nginx
systemctl enable nginx
