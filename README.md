# RotaFácil — Mobilidade Inteligente

**Projeto de conclusão de curso — Generation Brasil | AWS re/Start — Graduate**  
**Desafio final em equipe • Caso 1: Plataforma de transporte por aplicativo para mobilidade urbana**

**Equipe do projeto:** Richardson Firmino, Érica Claudiano, Mônica Ribeiro, Kauã Muniz, Ana Verduguez, Guilherme Moreira e Vítor Nazaré.

## Sumário

- [Sobre o projeto](#sobre-o-projeto)
- [Objetivo do desafio](#objetivo-do-desafio)
- [Regras de negócio](#regras-de-negócio)
  - [Requisitos funcionais](#requisitos-funcionais)
  - [Requisitos não funcionais](#requisitos-não-funcionais)
- [Problemas identificados na arquitetura atual](#problemas-identificados-na-arquitetura-atual)
- [Proposta de arquitetura AWS](#proposta-de-arquitetura-aws)
  - [Diagrama da arquitetura](#diagrama-da-arquitetura)
- [Componentes e serviços da arquitetura](#componentes-e-serviços-da-arquitetura)
- [Como a arquitetura responde aos problemas](#como-a-arquitetura-responde-aos-problemas)
- [Fluxo geral da arquitetura](#fluxo-geral-da-arquitetura)
- [Tecnologias e serviços considerados](#tecnologias-e-serviços-considerados)
- [Equipe do projeto](#equipe-do-projeto)
- [Conclusão](#conclusão)
- [Conclusão do curso e contexto do projeto](#conclusão-do-curso-e-contexto-do-projeto)
- [Status do projeto](#status-do-projeto)
- [Licença](#licença)

---

## Sobre o projeto

O RotaFácil é uma proposta de plataforma de mobilidade urbana que conecta passageiros a motoristas parceiros para corridas urbanas, com acompanhamento da localização em tempo real e cálculo de tarifa dinâmica.

Este projeto foi desenvolvido como parte do desafio de conclusão do programa **Generation Brasil**, no contexto do curso **AWS re/Start — Graduate**. O trabalho analisa os problemas de uma arquitetura legada e apresenta uma proposta de modernização utilizando serviços da Amazon Web Services (AWS), considerando disponibilidade, escalabilidade, segurança, monitoramento e otimização de custos.

> **Importante:** este repositório documenta uma proposta acadêmica de arquitetura. A presença de serviços no diagrama não significa que a infraestrutura tenha sido implantada ou testada em produção.

## Objetivo do desafio

Propor uma arquitetura em nuvem capaz de responder aos requisitos de uma plataforma de transporte por aplicativo, considerando a variação de demanda, a necessidade de respostas rápidas, a continuidade das corridas e a proteção dos dados dos usuários.

## Regras de negócio

### Requisitos funcionais

- O passageiro solicita uma corrida e recebe o motorista disponível mais próximo em poucos segundos.
- A localização do motorista é atualizada continuamente durante a corrida.
- A tarifa varia conforme a demanda do momento, podendo aumentar nos horários de pico.
- Passageiro e motorista avaliam um ao outro ao final da corrida.
- O pagamento é processado por meio de integração com um gateway de pagamento externo.

### Requisitos não funcionais

- A plataforma precisa lidar com variações de demanda ao longo do dia, especialmente nos horários de pico e nos finais de semana.
- A atualização da localização exige respostas rápidas para preservar a experiência do usuário.
- O aplicativo deve permanecer disponível durante uma corrida em andamento.
- Os dados de localização e pagamento são sensíveis e precisam de proteção adequada, considerando os requisitos aplicáveis da Lei Geral de Proteção de Dados (LGPD).

## Problemas identificados na arquitetura atual

O desafio descreve uma arquitetura legada com capacidade fixa, aplicação monolítica e poucos mecanismos de resiliência. Os principais problemas identificados são:

### 1. Capacidade fixa e ausência de distribuição automática de carga

- Servidores físicos hospedados em um datacenter alugado, com capacidade contratada fixa.
- Aplicação monolítica executada em poucos servidores, sem distribuição automática de carga.
- Possibilidade de sobrecarga dos servidores durante os horários de pico.
- Uma falha de servidor pode afetar o acesso ao aplicativo e a experiência dos usuários.

### 2. Banco de dados sem redundância e arquivos armazenados localmente

- Banco de dados relacional único, sem cópia de segurança em outro local, representando um ponto único de falha.
- Fotos de perfil e documentos dos motoristas armazenados nos discos locais dos servidores.
- Uma falha de servidor ou de armazenamento pode interromper o acesso aos arquivos e dificultar a recuperação dos dados.
- O armazenamento local dificulta o compartilhamento de arquivos e a expansão da aplicação para várias instâncias.

### 3. Ausência de monitoramento automatizado

- O cenário legado não descreve um sistema de monitoramento automatizado.
- A equipe só toma conhecimento de uma indisponibilidade quando um usuário reclama.
- A detecção tardia pode aumentar o tempo de resposta e o impacto da interrupção do serviço.

## Proposta de arquitetura AWS

A proposta utiliza recursos da AWS distribuídos em duas zonas de disponibilidade, com segmentação de rede pública e privada, distribuição de tráfego, banco de dados com implantação Multi-AZ, armazenamento de objetos e serviços de monitoramento e segurança.

### Diagrama da arquitetura

> **Imagem:** coloque o arquivo do diagrama na pasta `docs/` com o nome `arquitetura-aws.png`. Se o nome ou a localização forem diferentes, atualize o caminho abaixo.

![Diagrama da arquitetura AWS do RotaFácil](arquitetura-aws.png)

## Componentes e serviços da arquitetura

| Componente ou serviço | Função na proposta | Problema que ajuda a tratar |
|---|---|---|
| Amazon Route 53 | Serviço DNS para encaminhar o domínio ao endpoint configurado. | Organiza o acesso à aplicação por nome de domínio. |
| Amazon VPC e sub-redes públicas/privadas | Organizam e segmentam a rede da aplicação. | Ajudam a separar recursos expostos dos componentes internos e a controlar o tráfego. |
| Internet Gateway | Permite a conectividade entre a VPC e a internet, conforme as rotas configuradas. | Viabiliza o acesso externo aos recursos que precisam dele. |
| NAT Gateway | Permite que recursos em sub-redes privadas iniciem conexões de saída, conforme a configuração. | Evita a necessidade de expor diretamente instâncias privadas à internet para esse tipo de acesso. |
| Amazon EC2 — front-end | Hospeda a camada de apresentação da aplicação. | Separa a camada visual dos componentes de processamento. |
| Amazon EC2 — back-end | Executa o processamento das solicitações e a comunicação com os serviços autorizados. | Organiza a camada de processamento da aplicação. |
| Application Load Balancer (ALB) | Distribui requisições entre os destinos configurados. | Reduz a concentração de tráfego em uma única instância. |
| Auto Scaling | Ajusta a quantidade de instâncias conforme políticas e métricas configuradas. | Ajuda a lidar com variações de demanda e horários de pico. |
| Duas zonas de disponibilidade | Distribuem recursos entre zonas distintas, conforme a implantação. | Ajudam a reduzir a dependência de uma única zona. |
| Amazon RDS Multi-AZ | Mantém uma implantação de banco de dados relacional com instância principal e standby, conforme a configuração do serviço. | Melhora a disponibilidade do banco de dados diante de determinadas falhas. |
| Backups do banco de dados | Criam cópias de segurança para recuperação. | Ajudam a recuperar dados em caso de exclusão, corrupção ou falha. |
| Amazon S3 | Armazena objetos, como fotos e documentos, de forma centralizada. | Reduz a dependência de arquivos guardados apenas nos discos das instâncias EC2. |
| S3 Intelligent-Tiering | Pode otimizar os custos de armazenamento de acordo com os padrões de acesso. | Ajuda na gestão de custos quando o padrão de acesso é adequado. |
| AWS Lambda | Executa funções sob demanda para tarefas definidas na solução. | Permite automatizar tarefas sem manter um servidor dedicado para cada função. |
| Amazon CloudWatch | Coleta métricas e logs e permite configurar alarmes. | Ajuda a detectar anomalias e falhas com maior antecedência. |
| Amazon SNS | Distribui notificações para assinantes configurados. | Apoia o envio de alertas para a equipe ou outros componentes. |
| AWS KMS | Gerencia chaves criptográficas. | Apoia a proteção de dados por criptografia quando integrado aos serviços pertinentes. |
| Amazon Macie | Ajuda a identificar dados sensíveis em fontes compatíveis, como o Amazon S3. | Apoia a identificação e a proteção de informações confidenciais armazenadas. |
| AWS Certificate Manager (ACM) | Gerencia certificados TLS/SSL para serviços integrados compatíveis. | Apoia o uso de conexões criptografadas nos endpoints compatíveis. |
| Grupos de segurança | Controlam o tráfego de entrada e saída dos recursos associados. | Ajudam a restringir a comunicação aos acessos permitidos. |
| Gateway de pagamento externo | Representa a integração de pagamento indicada no desafio. | Considera o processamento por um provedor externo, cujo fornecedor não foi especificado. |

**Nota técnica:** esta documentação descreve a proposta de arquitetura apresentada para o desafio. A presença de um serviço no diagrama não comprova que ele esteja implantado ou configurado em produção. Os benefícios dependem das configurações, das políticas de segurança e dos testes realizados.

## Como a arquitetura responde aos problemas

### Distribuição de carga e escalabilidade

O Application Load Balancer distribui o tráfego entre os destinos configurados. O Auto Scaling permite ajustar a quantidade de instâncias de acordo com políticas e métricas definidas. Em conjunto, esses recursos podem ajudar a lidar com o aumento da demanda nos horários de pico.

### Disponibilidade e recuperação de dados

O Amazon RDS Multi-AZ mantém uma implantação standby em outra zona de disponibilidade, conforme a configuração do serviço, e pode apoiar a continuidade do banco de dados diante de determinadas falhas. Os backups são um mecanismo complementar de recuperação: alta disponibilidade e backup não são a mesma coisa.

### Armazenamento de fotos e documentos

O Amazon S3 permite centralizar fotos de perfil e documentos, reduzindo a dependência dos discos locais das instâncias EC2. As opções de armazenamento, as políticas de acesso e os mecanismos de proteção devem ser definidos conforme os requisitos do projeto.

### Monitoramento e alertas

O Amazon CloudWatch pode monitorar métricas e logs e disparar alarmes. O Amazon SNS pode distribuir notificações para assinantes configurados. O AWS Lambda pode executar ações automatizadas quando integrado ao fluxo definido.

### Segurança e proteção de dados

A segmentação de rede, os grupos de segurança, o gerenciamento de chaves pelo AWS KMS, a análise de dados sensíveis com o Amazon Macie e o uso de certificados TLS/SSL em serviços compatíveis contribuem para uma estratégia de segurança em camadas.

A conformidade com a LGPD também exige decisões sobre coleta, finalidade, retenção, acesso e tratamento dos dados; ela não depende apenas dos serviços de nuvem utilizados.

## Fluxo geral da arquitetura

1. Passageiros e motoristas acessam a plataforma pela internet.
2. O Amazon Route 53 resolve o domínio, quando configurado.
3. O tráfego chega ao ponto de entrada da aplicação e ao Application Load Balancer, conforme o desenho e as configurações propostas.
4. O balanceador encaminha as requisições aos destinos configurados nas zonas de disponibilidade.
5. O back-end comunica-se com o banco de dados e com os serviços auxiliares autorizados.
6. O Amazon S3 pode armazenar fotos e documentos.
7. O CloudWatch acompanha métricas e logs; alarmes podem acionar notificações pelo SNS e funções Lambda, conforme as integrações definidas.
8. O pagamento é direcionado à integração com o gateway externo previsto no desafio.

Este fluxo é uma explicação conceitual do diagrama, não uma afirmação de que as funcionalidades já foram implementadas.

## Tecnologias e serviços considerados

- Amazon Web Services (AWS)
- Amazon EC2
- Amazon VPC e sub-redes públicas e privadas
- Internet Gateway e NAT Gateway
- Application Load Balancer (ALB)
- Auto Scaling
- Amazon RDS Multi-AZ
- Amazon S3 e S3 Intelligent-Tiering
- AWS Lambda
- Amazon CloudWatch
- Amazon SNS
- AWS KMS
- Amazon Macie
- Amazon Route 53
- AWS Certificate Manager (ACM)
- Grupos de segurança
- Git e GitHub para versionamento e documentação

As linguagens de programação, os frameworks e o fornecedor do gateway de pagamento não foram especificados no material fornecido e, por isso, não são presumidos nesta documentação.

## Equipe do projeto

- Richardson Firmino
- Érica Claudiano
- Mônica Ribeiro
- Kauã Muniz
- Ana Verduguez
- Guilherme Moreira
- Vítor Nazaré

## Conclusão

O desafio do RotaFácil consistiu em analisar as limitações de uma arquitetura legada e propor uma infraestrutura em nuvem com foco em distribuição de carga, escalabilidade, disponibilidade, armazenamento centralizado, monitoramento e segurança.

A proposta combina serviços gerenciados e recursos de computação da AWS para responder aos problemas apresentados no caso. A efetividade da solução deve ser validada por meio de implementação, testes de carga, testes de recuperação, revisão das configurações de segurança e avaliação de custos.

Como entrega de conclusão do programa Generation Brasil, no contexto do curso AWS re/Start — Graduate, este trabalho reúne a análise do cenário proposto, a identificação dos problemas da arquitetura legada e a elaboração, em equipe, de uma proposta de arquitetura AWS para o RotaFácil. A atividade permitiu aplicar conceitos de computação em nuvem, disponibilidade, escalabilidade, monitoramento e segurança em um estudo de caso de mobilidade urbana.

## Conclusão do curso e contexto do projeto

Este trabalho foi desenvolvido como desafio de conclusão do programa **Generation Brasil**, no contexto do curso **AWS re/Start — Graduate**. O projeto aplica conceitos de computação em nuvem e arquitetura AWS a um estudo de caso de mobilidade urbana.

## Status do projeto

**Status:** projeto acadêmico de arquitetura e documentação.  
A documentação apresenta uma proposta conceitual; atualize esta seção caso os recursos sejam implantados e testados.

## Licença

A licença ainda não foi definida. A equipe deve decidir os termos de reutilização do código e dos materiais antes de adicionar um arquivo de licença.
