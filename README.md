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

