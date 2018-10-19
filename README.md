# Continuous Deployment usando Docker


## Contextualização

Muitas instituições usam ferramentas (Jenkins, TFS, etc) para automatizar as fases de publicação de um software. Nelas, normalmente ficam informações pertinentes ao projeto como "comando (tasks) para baixar dependências, compilar, testar, etc" assim como a tecnologia envolvida como "JAVA, .NET, Node, etc".

Muitas vezes este método funciona bem, porém exige a necessidade de que uma equipe (muitas vezes diferente) faça todo o papel de se configurar a infraestrutura necessária para que cada etapa funcione tais como "máquinas virtuais onde será publicado o software (servidor de aplicação)", "configuração na ferramenta de automação (criação dos comandos) para o software", entre outras particularidades da aplicação para o seu ambiente.

Em um cenário onde as aplicações estão ficando cada vez mais difundidas e pequenas (microserviços), isto cria uma demanda muito grande para criação de todo esse processo para cada peça de software. Aliado ao fato de que as demandas por resultado de TI (especialmente criação e desenvolvimento de soluções) são cada vez mais velozes, faz com que se busquemos meios para facilitar e/ou aprimorar toda essa etapa (criação da automação).


## Objetivo (Vantagens)

Utilizar docker no desenvolvimento pode proporcionar múltiplas vantagens, porém, nem sempre, estas são utilizadas.

Seguem alguns pontos onde o Docker facilita em todo este processo:
- Criação do processo de compilação do software (Dockerfile multi-stage)
- Criação do processo de execução do teste automatizado (Dockerfile + docker-compose)
- Criação do processo de publicação da aplicação (docker-compose)
