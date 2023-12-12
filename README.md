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
```
touch main.tf
touch variables.tf
mkdir modules
cd modules
mkdir instances
cd instances
touch instances.tf
touch outputs.tf
touch variables.tf
cd ..
mkdir storage
cd storage
touch storage.tf
touch outputs.tf
touch variables.tf
cd ~/
```
2. Preencha os arquivos ```variables.tf``` no diretório raiz e nos módulos. Adicione três variáveis para cada arquivo: ```region```, ```zone``` e ```project_id```. Como valores padrão, use ```region informado no Lab```, ```zone informado no Lab``` e seu ID do projeto do Google Cloud.
```
# arquivo variables.tf colocar no três arquivos.
variable "project_id" {
  description = "The project ID to host the network in"
  default     = "ID informado no Lab"
}

variable "zone" {
  description = "The name of the region used"
  default     = "zone informado no Lab"
}

variable "region" {
  description = "The name of the zone used"
  default     = "Region informado no Lab"
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
  version = "3.55.0"
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
> ```
> terraform init
>  ```
> - Em seguida, escreva as configurações do recurso no arquivo instances.tf para corresponder às instâncias atuais.
> - Em seguida, escreva as configurações do [recurso](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance)  no arquivo ```instances.tf``` para corresponder às instâncias atuais.
> >    -  Dê às instâncias os nomes ```tf-instance-1``` e ```tf-instance-2```.
> >    -  Neste laboratório, a configuração do recurso deve ser mínima. Para isso, você vai precisar incluir estes argumentos extras na configuração: ```machine_type```, ```boot_disk```, ```network_interface```, ```metadata_startup_script``` e ```allow_stopping_for_update```. No caso dos últimos dois argumentos, use a configuração a seguir para que não seja necessário recriá-la:
> > ```
> > metadata_startup_script = <<-EOT
> >         #!/bin/bash
> >     EOT
> > allow_stopping_for_update = true
> > ```
> ```
> #Arquivo instances.tf
> 
> resource "google_compute_instance" "tf-instance-1" {
>    name = "tf-instance-1"
>    machine_type = "e2-micro"
>    zone = var.zone
>
>    boot_disk {
>        initialize_params {
>        image = "debian-cloud/debian-11"
>        }
>    }
> 
>    network_interface {
>      network = "default"
>   }
>    metadata_startup_script = <<-EOT
>            #!/bin/bash
>        EOT
>    allow_stopping_for_update = true
>}
>
> resource "google_compute_instance" "tf-instance-2" {
>    name = "tf-instance-2"
>    machine_type = "e2-micro"
>    zone = var.zone
>
>   boot_disk {
>        initialize_params {
>        image = "debian-cloud/debian-11"
>        }
>    }
>
>    network_interface {
>      network = "default"
>   }
>    metadata_startup_script = <<-EOT
>            #!/bin/bash
>        EOT
>    allow_stopping_for_update = true
>}
> ```
> - Depois de escrever as configurações do recurso no módulo, use o comando ```terraform import``` e importe essas preferências para o módulo ```instances```.
> 
> O arquivo instances.tf deverá ficar assim.
> ```
> Depois de preencher o arquivo instances.tf Execute o comando para importar as instancias
> ```
> terraform import module.instances.google_compute_instance.tf-instance-1 < Cole aqui o instance id 1>
> terraform import module.instances.google_compute_instance.tf-instance-2 < Cole aqui o instance id 2>
> ``` 
3. Aplique as alterações. Como você não preencheu todos os argumentos na configuração, o código ```apply``` vai ***atualizar as instâncias atuais***. Essa opção é aceita no laboratório, mas é necessário preencher todos os argumentos corretamente antes da importação em um ambiente de produção.
> Execute o comando
> ```
> terraform plan
> terraform apply
> ```
## Tarefa 3: configurar um back-end remoto
1. Crie um [recurso de bucket do Cloud Storage](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/storage_bucket) no módulo ```storage```. Para ***nomear*** o bucket, use ***Bucket Name***. Para os outros argumentos, use estes valores:
>> - location = "US"
>> - force_destroy = true
>> - uniform_bucket_level_access = true
> ```
> # Arquivo storage.tf
> resource "google_storage_bucket" "bucket" {
>   name = "< O nome do bucket fornecido no Lab >"
>   location = "US"
>   force_destroy = true
>   uniform_bucket_level_access = true
> }
> ```
> 2. Adicione a referência do módulo ao arquivo ```main.tf```. Inicialize o módulo e aplique (```apply```) as mudanças para criar o bucket usando o Terraform.
> ```
> #Colocar no final do arquivo main.tf
> module "storage" {
> 	 source = "./modules/storage"	
> }
> ```
>> ***Nota***: Também é possível adicionar valores de saída ao arquivo ```outputs.tf```.
> ```
> # Execute os comandos
> terraform init
> terraform plan
> terraform apply
> ```
> 3. Configure o bucket de armazenamento como o [back-end remoto](https://www.terraform.io/docs/language/settings/backends/gcs.html) no arquivo ```main.tf```. Para possibilitar a avaliação, use o ***prefixo*** ```terraform/state```.
> ```
> No arquivo main.tf colocar entre as chaves do parametro terraform {  ...  }
> terraform {
>
>  # ...
>  backend "gcs" {
>    bucket  = "< O nome do bucket fornecido no Lab >"
>    prefix  = "terraform/state"
>  }
> # ...
> }
> ```
> 4. Se a configuração estiver correta, após o comando ```init```, o Terraform vai perguntar se você quer copiar os dados de estado para o novo back-end. Digite ```yes``` no prompt.
> ```
> # Digite o comando
> terraform init
> ```
## Tarefa 4: modificar e atualizar a infraestrutura
> 1. Acesse o módulo ```instances``` e modifique o recurso ***tf-instance-1*** para usar um tipo de máquina ```Verificar o modelo no lab```.
> ```
> machine_type = "copie o modelo indicado no lab"
> ```
> 2. Altere o recurso ***tf-instance-2*** para usar um tipo de máquina ```Verificar o modelo no lab```.
> ```
> machine_type = "copie o modelo indicado no lab"
> ```
> 3 Adicione um terceiro recurso de instâncias chamado ***Instance Name***. Use um tipo de máquina ```Verificar o modelo no lab``` para ele.
> ```
> # Coloque no final do arquivo instances.tf
> 
> resource "google_compute_instance" "tf-instance-3"{
>   name         = "< Nome fornecido no lab >"
>   machine_type = "copie o modelo indicado no lab"
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
> 4 Inicialize o Terraform e aplique (```apply```) as mudanças.
> ```
> terraform init
> terraform apply
> ```
>> Nota: Como alternativa, adicione valores de saída aos recursos no arquivo ```outputs.tf``` do módulo.
## Tarefa 5: usar taint e destruir recursos
> 1. [Use taint](https://www.terraform.io/docs/cli/commands/taint.html) na terceira instância ***Instance Name***. Em seguida, planeje (```plan```) e aplique (```apply```) as mudanças para recriá-la.
> ```
> terraform taint module.instances.google_compute_instance.tf-instance-3
> ```
> terraform plan
> terraform init
> ```
> 3. Remova o recurso do arquivo de configuração para destruir a terceira instância ***tf-instance-3***. Depois disso, inicialize o Terraform e aplique (```apply```) as mudanças.
>> Apague as referências da ***Instance Name*** do módulo ```instances.tf```.
## Tarefa 6: usar um módulo do Registry
> 1. No Terraform Registry, procure o [Módulo de rede](https://registry.terraform.io/modules/terraform-google-modules/network/google/3.4.0).
> 2. Adicione o módulo ao arquivo ```main.tf```. Use as configurações a seguir:
>> - Utilize a versão ```3.4.0```. Outras versões podem gerar erros de compatibilidade.
>> - Nomeie a VPC como ```VPC Name``` e use o modo de roteamento ***global***.
>> - Especifique ***duas*** sub-redes na região ```us-east1``` chamadas de ```subnet-01``` e ```subnet-02```. Para os argumentos de sub-rede, é necessário incluir um ***Nome***, um ***IP*** e uma ***Região***.
>> - Use o IP ```10.10.10.0/24``` para ```subnet-01``` e ```10.10.20.0/24``` para ```subnet-02```.
>> - Como essa VPC ***não*** exige intervalos secundários ou rotas associadas, omita essas opções da configuração.
>> ```
>> #Colocar no final do arquivo main.tf
>> 
>>  module "vpc" {
>>   source        = "terraform-google-modules/network/google"
>>   version      = "~> 3.4.0"
>>   project_id   = var.project_id
>>   network_name = "< Nome fornecido no lab >"
>>   routing_mode = "GLOBAL"
>>   mtu          = 1460
>> 
>>   subnets = [
>>     {
>>       subnet_name   = "subnet-01"
>>       subnet_ip     = "10.10.10.0/24"
>>       subnet_region = var.region
>>     },
>>     {
>>       subnet_name   = "subnet-02"
>>       subnet_ip     = "10.10.20.0/24"
>>       subnet_region = var.region
>>       subnet_private_access = "true"
>>       subnet_flow_logs = "true"
>>     }
>>   ]
>> }
>> ``` 
> 3. Depois de escrever a configuração do módulo, inicialize o Terraform e execute a aplicação (```apply```) para criar as redes.
> > ```
> > terraform init
> > terraform apply
> > ``` 
> 4. Em seguida, acesse o arquivo ```instances.tf``` e atualize os recursos de configuração para conectar ***tf-instance-1*** a ```subnet-01``` e ***tf-instance-2*** a ```subnet-02```.
>>   ***Nota***: nessa configuração de instância, você vai precisar atualizar o argumento de ***rede*** para ```VPC Name``` e adicionar o argumento de ***sub-rede*** com a sub-rede certa para cada instância.
>> ```
>> network_interface {
>>   network = "< Nome fornecido no lab >"
>>   subnetwork = "Para a instacia ***tf-instance-1*** coloque ***subnet-01*** e Para a instacia ***tf-instance-2*** coloque ***subnet-02***"
>> }
>> ```
## Tarefa 7: configurar um firewall
> 1. Crie um recurso de [regra de firewall](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_firewall) chamado ***tf-firewall*** no arquivo ```main.tf```.
> > - Essa regra de firewall deve permitir que a rede ```VPC Name``` autorize conexões de entrada em todos os intervalos de IP (```0.0.0.0/0```) na ***porta TCP 80***.
> > - Adicione o argumento ```source_ranges``` com o intervalo de IP correto (```0.0.0.0/0```).
> > - Inicialize o Terraform e aplique (```apply```) as mudanças.
> > ***Nota***: para recuperar o argumento ```network``` obrigatório, inspecione o estado e encontre o ***ID*** ou o ***self_link*** do recurso ```google_compute_network``` que você criou. Essa informação fica no formulário ```projects/PROJECT_ID/global/networks/VPC Name```.
> ```
> # Colocar no arquivo main.tf
> 
> resource "google_compute_firewall" "tf-firewall" {
>  name    = "tf-firewall"
>  network = "projects/< Projeto fornecido no lab >/global/networks/< nome da VPC fornecida no lab >"
>
>  allow {
>    protocol = "icmp"
>  }
>
>  allow {
>    protocol = "tcp"
>    ports    = ["80"]
>  }
>  source_tags = ["web"]
>  source_ranges = ["0.0.0.0/0"]
>}
> ```
> > ```
> > terraform init
> > terraform apply
> > ```
## Teste de conectividade (opcional)
> Depois de criar uma regra de firewall para permitir conexões internas na VPC, é possível executar um teste de conectividade de rede.
> 1. As duas VMS precisam estar em execução.
> 2. Acesse ***Network Intelligence > Testes de conectividade***. Faça o teste nas duas VMs para verificar se elas são acessíveis. Depois disso, a conectividade entre as instâncias estará validada.
> > ***Nota***: veja se a ***API Network Management está ativada***. Caso não esteja, clique em ***Ativar***.
> Preencha a configuração, como abaixo:
> > Test name: terrform-network-check
> > Protocol: tcp
> > Source
> > > Source endpoint: VM Instance
> > > Source VM Instance: tf-instance-1
> > Destination
> > > Source endpoint: VM Instance
> > > Source VM Instance: tf-instance-2
> > Destination port: 80
> > Clique em create
