# Continuous Deployment usando Docker


## Contextualização

Muitas instituições usam ferramentas (Jenkins, TFS, etc) para automatizar as fases de publicação de um software. Nelas, normalmente ficam informações como "comando (tasks) para baixar dependências, compilar, testar, publicar, etc" assim como configurações pertinentes a tecnologia do projeto ("JAVA, .NET, Node, etc").

Muitas vezes este método funciona bem, porém exige a necessidade de que uma equipe (muitas vezes diferente) faça todo o papel de se configurar a infraestrutura necessária para que cada etapa funcione tais como "máquinas virtuais onde será publicado o software (servidor de aplicação)", "configuração na ferramenta de automação (criação dos comandos) para o software", entre outras particularidades da aplicação para o seu ambiente.

Em um cenário onde as aplicações estão ficando cada vez mais difundidas e pequenas (microserviços), isto cria uma demanda muito grande para criação de todo esse processo para cada peça de software. Aliado ao fato de que as demandas por resultado de TI (especialmente criação e desenvolvimento de soluções) são cada vez mais velozes, faz com que busquemos meios para facilitar e/ou aprimorar toda essa etapa (criação da automação).


## Objetivo (Vantagens)

Utilizar docker no desenvolvimento pode proporcionar múltiplas vantagens, porém, nem sempre, estas são utilizadas.

Considere que Docker de forma geral é uma tecnologia de criação/execução de imagens (algo como uma template de máquina virtual) e criação/execução de ambientes.

> Normalmente, a utilização do docker é vista somente para a publicação do software. É o típico cenário em que o desenvolvedor copia "somente o binário" (já compilado em sua máquina ou na ferramenta de automação) para dentro da imagem e publica esta. Porém, faz com que "a máquina do desenvolvedor ou a ferramenta de automação necessitem das ferramentas de desenvolvimento instaladas" e consequentemente alguém (ou equipe) para gerenciar essa infraestrutura (no caso da ferramenta de automação), além de que cria um acoplamento nesta (a partir do momento em que se cria nela a configuração/execução das etapas necessárias).

Seguem alguns pontos onde o Docker facilita em todo este processo:
- Criação do processo de compilação do software (via Dockerfile multi-stage)
  > Isto permite que o mesmo Dockerfile que é utilizado para se compilar a aplicação, seja utilizado na ferramenta de automação
- Criação do processo de execução do teste automatizado (via Dockerfile multi-stage)
  > Isto permite que o mesmo Dockerfile que é utilizado para se compilar a aplicação, também opcionalmente faça a execução dos testes automatizados (Unitários ou de Integração) 
- Criação do processo de publicação da aplicação (docker-compose)
  > Permite que seja descrito (de forma declarativa) como deve ser criado o ambiente
- Independência da aplicação quanto a suas fronteiras (docker-compose)
  > Permite que a configuração de integrações/fronteiras seja feito no arquivo de configuração do ambiente (docker-compose), explicitando suas dependências/integrações
