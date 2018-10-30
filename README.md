# Continuous Deployment usando Docker


## Contextualização

Muitas instituições usam ferramentas (Jenkins, TFS, etc) para automatizar as fases de publicação de um software. Nelas, normalmente ficam informações como "comando (tasks) para baixar dependências, compilar, testar, publicar, etc" assim como configurações pertinentes a tecnologia do projeto ("JAVA, .NET, Node, etc").

Muitas vezes este método funciona bem, porém exige a necessidade de que uma equipe (muitas vezes diferente) faça todo o papel de se configurar a infraestrutura necessária para que cada etapa funcione tais como "máquinas virtuais onde será publicado o software (servidor de aplicação)", "configuração na ferramenta de automação (criação dos comandos) para o software", entre outras particularidades da aplicação para o seu ambiente.

Em um cenário onde as aplicações estão ficando cada vez mais difundidas e pequenas (microserviços), isto cria uma demanda muito grande para criação de todo esse processo para cada peça de software. Aliado ao fato de que as demandas por resultado de TI (especialmente criação e desenvolvimento de soluções) são cada vez mais velozes, faz com que busquemos meios para facilitar e/ou aprimorar toda essa etapa (criação da automação).


## Objetivo (Vantagens)

Utilizar docker no desenvolvimento pode proporcionar múltiplas vantagens, porém, nem sempre, estas são utilizadas.

_Considere que Docker de forma geral é uma tecnologia de criação/execução de imagens (algo como uma template de máquina virtual) e criação/execução de ambientes._

> Normalmente, a utilização do docker é vista somente para a publicação do software. É o típico cenário em que o desenvolvedor copia "somente o binário" (já compilado em sua máquina ou na ferramenta de automação) para dentro da imagem e publica esta. Porém, faz com que "a máquina do desenvolvedor ou a ferramenta de automação necessitem das ferramentas de desenvolvimento instaladas" e consequentemente alguém (ou equipe) para gerenciar essa infraestrutura (no caso da ferramenta de automação), além de que cria um acoplamento nesta (a partir do momento em que se cria nela a configuração/execução das etapas necessárias).

Seguem alguns pontos onde o Docker facilita em todo este processo:
- Criação do processo de compilação do software (via Dockerfile multi-stage)
  > Permite que o mesmo Dockerfile que é utilizado para se compilar a aplicação, seja utilizado na ferramenta de automação
- Criação do processo de execução do teste automatizado (via Dockerfile multi-stage)
  > Permite que o mesmo Dockerfile que é utilizado para se compilar a aplicação, também opcionalmente faça a execução dos testes automatizados (Unitários ou de Integração) 
- Criação do processo de publicação da aplicação (docker-compose)
  > Permite que seja descrito (de forma declarativa) como deve ser criado o ambiente
- Explicitação da aplicação quanto a suas fronteiras (docker-compose)
  > Permite que a configuração de integrações/fronteiras seja feito no arquivo de configuração do ambiente (docker-compose), explicitando suas dependências/integrações


## Como Funciona?

> `Dockerfile`: [(Referência)](https://docs.docker.com/engine/reference/builder/)
>
> `docker-compose.yml`: [(Referência)](https://docs.docker.com/compose/compose-file/)

O repositório GIT deve conter os seguintes arquivos:
- `Dockerfile` - (Exemplo)[./dockerfile.md]
  > Responsável por criar a imagem, compilar o código, executar os testes e executar a aplicação.
- `docker-compose.build.yml`
  > Responsável por conter os argumentos necessários para criar a imagem usando o `Dockerfile`.
- `docker-compose.ci.yml`
  > Responsável por conter os argumentos e serviços necessários para executar os testes de integração _(que necessitam de um ambiente completo para execução dos testes - ex: banco de dados)_.
- `docker-compose.debug.yml`
  > Responsável por conter os argumentos necessários para executar a aplicação e expor uma porta em que a IDE irá utilizar para debug (attach).
- _`docker-compose.{environment}.yml`_
  > Responsável por conter as configurações (variáveis de ambiente, labels, etc) e imagem a ser utilizada para a publicação em um ambiente específico.
    >
    > Pode-se haver múltiplos arquivos que será utilizado pela ferramenta de automação para criar/publicar no ambiente específico.
    >
    > Exemplo:
    > - `docker-compose.alpha-a.yml`
    > - `docker-compose.beta.yml`
    > - `docker-compose.rc.yml`
    > - `docker-compose.stable.yml`
