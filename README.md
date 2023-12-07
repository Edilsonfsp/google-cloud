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
# arquivo variables.tf da raíz.

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
```
# Arquivo main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}

provider "google" {
  version = "3.5.0"
  project = var.project_id
  region  = var.region
  zone    = var.zone
}


```
4. Inicialize o Terraform.
```
terraform init
```
## Tarefa 2: importar a infraestrutura
1. No Console do Google Cloud, no ***Menu de navegação***, clique em ***Compute Engine > Instâncias de VM***.
   Duas instâncias chamadas ```tf-instance-1``` e ```tf-instance-2``` já foram criadas para você.
> ***Nota***: ao clicar em uma das instâncias, é possível ver o ***ID da instância***, a ***imagem do disco de inicialização*** e o ***tipo de máquina***. Todas essas informações são necessárias para escrever corretamente as configurações e fazer a importação delas para o Terraform.
2. [Importe](https://www.terraform.io/docs/cli/commands/import.html#example-import-into-module) as ```instâncias``` atuais para o módulo instances. Para isso, siga estas etapas:
> - Primeiro, adicione a referência do módulo ao arquivo main.tf e reinicialize o Terraform.
> ```
> #Colocar no final do arquivo main.tf
> module "instances" {
> 	 source = "./modules/instances"	
> }
> ```
> - Em seguida, escreva as configurações do recurso no arquivo instances.tf para corresponder às instâncias atuais.
> ```
> # instances.tf
> resource "google_compute_instance" "instances"{ }
> ```
> - Em seguida, escreva as configurações do [recurso](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance)  no arquivo ```instances.tf``` para corresponder às instâncias atuais.
> >    -  Dê às instâncias os nomes ```tf-instance-1``` e ```tf-instance-2```.
> >    -  Neste laboratório, a configuração do recurso deve ser mínima. Para isso, você vai precisar incluir estes argumentos extras na configuração: ```machine_type```, ```boot_disk```, ```network_interface```, ```metadata_startup_script``` e ```allow_stopping_for_update```. No caso dos últimos dois argumentos, use a configuração a seguir para que não seja necessário recriá-la:
> > ```
> > metadata_startup_script = <<-EOT
> >         #!/bin/bash
> >     EOT
> > allow_stopping_for_update = true
> > ```
> - Depois de escrever as configurações do recurso no módulo, use o comando ```terraform import``` e importe essas preferências para o módulo ```instances```.
> Execute o comando
> ```
> terraform import google_compute_instance.instances {{tf-instance-1}}
> terraform import google_compute_instance.instances {{tf-instance-2}}
> ```
> Verifique se as instâncias foram importadas para o estado do Terraform:
> ```
> terraform show
> ```
> Copie o estado do Terraform para o arquivo instances.tf:
> ```
> terraform show -no-color > instances.tf
> ```
> O arquivo instances.tf deverá ficar assim.
> ```
># Arrume o arquivo instances.tf
> 
> resource "google_compute_instance" "tf-instance-1"{
>   name         = "tf-instance-1"
>   machine_type = "ver no lab"
>   zone         = var.zone
> 
>   boot_disk {
>     initialize_params {
>       image = "debian-cloud/debian-11"
>     }
>   }
> 
>   network_interface {
>     network = "default"
> 
>     access_config {
>       // Ephemeral public IP
>     }
>   }
> 
> metadata_startup_script = <<-EOT
>         #!/bin/bash
>     EOT
> allow_stopping_for_update = true
> }
> > 
> resource "google_compute_instance" "tf-instance-2"{
>   name         = "tf-instance-2"
>   machine_type = "ver no lab"
>   zone         = var.zone
> 
>   boot_disk {
>     initialize_params {
>       image = "debian-cloud/debian-11"
>     }
>   }
> 
>   network_interface {
>     network = "default"
> 
>     access_config {
>       // Ephemeral public IP
>     }
>   }
> 
> metadata_startup_script = <<-EOT
>         #!/bin/bash
>     EOT
> allow_stopping_for_update = true
> }
> ```
3. Aplique as alterações. Como você não preencheu todos os argumentos na configuração, o código ```apply``` vai ***atualizar as instâncias atuais***. Essa opção é aceita no laboratório, mas é necessário preencher todos os argumentos corretamente antes da importação em um ambiente de produção.
> Execute o comando
> ```
> terraform apply
> ```
## 3. Tarefa 3: configurar um back-end remoto
1. Crie um [recurso de bucket do Cloud Storage](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/storage_bucket) no módulo ```storage```. Para ***nomear*** o bucket, use ***Bucket Name***. Para os outros argumentos, use estes valores:
>> - location = "US"
>> - force_destroy = true
>> - uniform_bucket_level_access = true
> ```
> # Arquivo storage.tf
> resource "google_storage_bucket" "bucket for back-end remote" {
>   name = var.bucket-back-end
>   location = "US"
>   force_destroy = true
>   uniform_bucket_level_access = true
> }
> ```
> ```
> #Colocar no final do arquivo variables.tf da raíz.
> variable "bucket-back-end" {
>  description = "The name of the bucket for back end"
>  default     = "seunome-XXX" # ex: "edilson-523"
> }
> ```
> 2. Adicione a referência do módulo ao arquivo ```main.tf```. Inicialize o módulo e aplique (```apply```) as mudanças para criar o bucket usando o Terraform.
> ```
> #Colocar no final do arquivo main.tf
> module "bucket" {
> 	 source = "./modules/storage"	
> }
> ```
>> ***Nota***: Também é possível adicionar valores de saída ao arquivo ```outputs.tf```.
> ```
> # Execute os comandos
> terraform plan
> terraform apply
> ```
> 3. Configure o bucket de armazenamento como o [back-end remoto](https://www.terraform.io/docs/language/settings/backends/gcs.html) no arquivo ```main.tf```. Para possibilitar a avaliação, use o ***prefixo*** ```terraform/state```.
> ```
>
> ```
> Inicializar o
> ```
terraform init -migrate-state
> ```
4. Tarefa 4: modificar e atualizar a infraestrutura
> - Acesse o módulo ```instances``` e modifique o recurso ***tf-instance-1*** para usar um tipo de máquina ```e2-standard-2```.
> ```
>
> ```
> - Altere o recurso ***tf-instance-2*** para usar um tipo de máquina ```e2-standard-2```.
> ```
> 
> ```
> - Adicione um terceiro recurso de instâncias chamado ***Instance Name***. Use um tipo de máquina ```e2-standard-2``` para ele.
> ```
># Coloque no final do arquivo instances.tf
> 
> resource "google_compute_instance" "tf-instance-name"{
>   name         = "Instance name"
>   machine_type = "e2-standard-2"
>   zone         = var.zone
> 
>   boot_disk {
>     initialize_params {
>       image = "debian-cloud/debian-11"
>     }
>   }
> 
>   network_interface {
>     network = "default"
> 
>     access_config {
>       // Ephemeral public IP
>     }
>   }
> 
> metadata_startup_script = <<-EOT
>         #!/bin/bash
>     EOT
> allow_stopping_for_update = true
> }
> ```
> - Inicialize o Terraform e aplique (```apply```) as mudanças.
> ```
> terraform init
> terraform apply
> ```
>> Nota: Como alternativa, adicione valores de saída aos recursos no arquivo ```outputs.tf``` do módulo.
