# Curso Google Cloud Engineer 
## Escola SENAI Mauá - Projeto conclusão do curso.
### Autor: Edilson F Souza
[![NPM](https://img.shields.io/npm/l/react)](https://github.com/Edilsonfsp/google-cloud/blob/main/LICENSE)
# Cenário do desafio
Você faz estágio na área de engenharia da nuvem em uma nova startup. Como seu primeiro projeto, pediram que você criasse uma infraestrutura de maneira rápida e eficiente, além de gerar um mecanismo de acompanhamento para consultas e mudanças futuras. Você recebeu orientações para usar o Terraform na realização dessa tarefa.
Nesse projeto, você vai usar o [Terraform](https://www.terraform.io/) para criar, implantar e acompanhar a infraestrutura no servidor preferido da startup, o Google Cloud. Você também vai precisar importar algumas instâncias com problemas de gerenciamento para sua configuração e fazer as correções necessárias.
Neste laboratório, você vai usar o Terraform para importar e criar várias instâncias de VM, uma rede VPC com duas sub-redes e uma regra de firewall para a VPC que permita conexões entre as duas instâncias. Você também vai criar um bucket do Cloud Storage para hospedar seu back-end remoto.
## Conhecimentos avaliados:
- Importar a infraestrutura atual para a configuração do Terraform
- Criar e fazer referência aos seus módulos do Terraform
- Adicionar um back-end remoto à configuração
- Usar e implementar um módulo do Terraform Registry
- Provisionar novamente, destruir e atualizar a infraestrutura
- Testar a conectividade entre os recursos criados
## Tarefa 1: criar os arquivos de configuração
1. No Cloud Shell, crie seus arquivos de configuração do Terraform e uma estrutura de diretórios como esta:

```
main.tf
variables.tf
modules/
└── instances
    ├── instances.tf
    ├── outputs.tf
    └── variables.tf
└── storage
    ├── storage.tf
    ├── outputs.tf
    └── variables.tf
```
2. Preencha os arquivos ```variables.tf``` no diretório raiz e nos módulos. Adicione três variáveis para cada arquivo: ```region```, ```zone``` e ```project_id```. Como valores padrão, use ```us-east1```, ```us-east1-c``` e seu ID do projeto do Google Cloud.
```
# arquivo variables.tf

variable "project_id" {
  description = "The project ID to host the network in"
  default     = "FILL IN YOUR PROJECT ID HERE"
}

variable "zone" {
  description = "The name of the region used"
  default     = "us-east1-c"
}

variable "region" {
  description = "The name of the zone used"
  default     = "us-east1"
}
```
> Nota: Use essas variáveis nas configurações do recurso sempre que possível.
3. Adicione o bloco do Terraform e o [Provedor do Google](https://registry.terraform.io/providers/hashicorp/google/latest/docs) ao arquivo ```main.tf```.  
  Verifique se o argumento de ***zona*** foi adicionado com os argumentos de ***projeto*** e ***região*** no bloco do provedor do Google.
4. Inicialize o Terraform.
```
terraform init
```
## Tarefa 2: importar a infraestrutura
1. No Console do Google Cloud, no ***Menu de navegação***, clique em ***Compute Engine > Instâncias de VM***.
   Duas instâncias chamadas ```tf-instance-1``` e ```tf-instance-2``` já foram criadas para você.
> ***Nota***: ao clicar em uma das instâncias, é possível ver o ***ID da instância***, a ***imagem do disco de inicialização*** e o ***tipo de máquina***. Todas essas informações são necessárias para escrever corretamente as configurações e fazer a importação delas para o Terraform.
2. [Importe](https://www.terraform.io/docs/cli/commands/import.html#example-import-into-module) as ```instâncias``` atuais para o módulo instances. Para isso, siga estas etapas:
- Primeiro, adicione a referência do módulo ao arquivo main.tf e reinicialize o Terraform.
```
module "instances" {
	source = "./edilson/modules/instances"
	
}
```
```
# instances.tf
resource "google_compute_instance" "instances"{ }
```
Execute o comando
```
terraform import google_compute_instance.instances {{tf-instance-1, tf-instance-2}}
```
Verifique se o contêiner foi importado para o estado do Terraform:
```
terraform show
```
Copie o estado do Terraform para o arquivo instances.tf:
```
terraform show -no-color > instances.tf
```

- Em seguida, escreva as configurações do [recurso](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance)  no arquivo ```instances.tf``` para corresponder às instâncias atuais.
    -  Dê às instâncias os nomes ```tf-instance-1``` e ```tf-instance-2```.
    -  Neste laboratório, a configuração do recurso deve ser mínima. Para isso, você vai precisar incluir estes argumentos extras na configuração: ```machine_type```, ```boot_disk```, ```network_interface```, ```metadata_startup_script``` e ```allow_stopping_for_update```. No caso dos últimos dois argumentos, use a configuração a seguir para que não seja necessário recriá-la:
```
metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
allow_stopping_for_update = true
```

```

```
