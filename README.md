# project-CI

Este repositório tem como objetivo armazenar um código de Pipeline CI que automatiza todo o processo de integração e entrega contínua. Ele realiza as seguintes etapas:
- Build do projeto
- Execução de testes
- Deploy da aplicação
- Análise de qualidade de código via SonarQube
- Armazenamento do artefato no Nexus
- Notificação de conclusão via Slack

O esquema abaixo ilustra melhor esse fluxo:
[![Ci Diagram](https://project-ci-repository.s3.sa-east-1.amazonaws.com/DiagramaCI.png "Ci Diagram")](https://project-ci-repository.s3.sa-east-1.amazonaws.com/DiagramaCI.pnghttp:// "Ci Diagram")

Usaremos toda a estrutura da AWS para montarmos nossa pipeline, incluindo três servidores EC2. 

[![AWS Diagram](https://project-ci-repository.s3.sa-east-1.amazonaws.com/DiagramaAWS.png "AWS Diagram")](https://project-ci-repository.s3.sa-east-1.amazonaws.com/DiagramaAWS.png "AWS Diagram")

### Inicialização e configuração dos servidores.
Para podermos configurar os três servidores, basta ir no diretório** /Setups** e executar o script de inicialização de cada uma dentro da AWS. É importante dizer que servidor do SonarQube e Jenkins serão inicializados em uma imagem do Ubuntu Server 24, enquanto o Nexus em um Amazon Linux 23.,