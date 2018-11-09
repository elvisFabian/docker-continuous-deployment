# Dockerfile

## Objetivo

Criar o arquivo [Dockerfile](https://docs.docker.com/engine/reference/builder/), o qual é responsável por compilar, testar, executar o projeto (opcionalmente debugando).

## Padrões

- Deve expor os resultados dos testes no caminho da variável `${OUTPUT_TEST_RESULTS}` (Padrão: `/TestResults`)
- Deve possíbilitar o Debug pela principal IDE da tecnologia (JAVA=Eclipse, .NET=Visual Studio, NODE=VSCode) mais o VSCode
- Os pacotes devem ser baixados em uma camada (layer) do docker e somente refeita quando solicitado (Ex: usando o argumento `--no-cache`)
- Estágio de Compilação deve ser nomeado `build`
- Estágio de Execução deve ser nomeado `final`
- Deve possibilitar informar o Registry do gerenciador de bibliotecas (MAVEN, NUGET, NPM) via argumento. (`NUGET_REGISTRY_{ID}`, `MAVEN_REGISTRY_{ID}`, `NPM_REGISTRY_{ID}`)
- Deve possibilitar informar o Proxy

## Exemplos

### JAVA

```dockerfile
# Imagem usada para a fase de construção (restaurar, compilar, testar)
FROM maven:3.5.4-jdk-8 AS build
WORKDIR /src
EXPOSE 80 443

# Argumentos necessários para a compilação do projeto
ARG SKIP_TEST=true

ARG JAVA_OPTS
ENV JAVA_OPTS=$JAVA_OPTS

# Instalar e configurar ferramentas


# Restaurar os pacotes
COPY pom.xml .
RUN mvn package -Dmaven.test.skip=true -Dspring-boot.repackage.skip=true

# Compilar o projeto
COPY . .
RUN mvn package -Dmaven.test.skip.exec=$SKIP_TEST
RUN cp target/*.jar ./app.jar

# Executar os testes
ENTRYPOINT mvn test

# Imagem usada para a fase de execução (executar)
FROM openjdk:8-jre AS final
WORKDIR /app
COPY --from=build /src/app.jar /app
ENTRYPOINT exec java $JAVA_OPTS -jar app.jar
EXPOSE 80 443
```

### .NET

```dockerfile
# Imagem usada para a fase de construção (restaurar, compilar, testar)
FROM microsoft/dotnet:2.1-sdk AS build
WORKDIR /src
EXPOSE 80 443

# Argumentos necessários para a compilação do projeto
ARG RUN_TEST=false

# Instalar e configurar ferramentas


# Restaurar os pacotes
COPY . .
dotnet restore --source ${NUGET_REGISTRY}

## Desta forma é feito o cache dos pacotes, porém precisa informar o projeto ()
# COPY Project/Project.csproj Project/
# dotnet restore --source ${NUGET_REGISTRY}

# Compilar o projeto
RUN dotnet publish /src/sistema-api.tjmt.jus.br/sistema-api.tjmt.jus.br.csproj -c ${CONFIGURATION} -o /app --no-build
RUN cp /app/*.dll ./app.dll

# Executar os testes
ENTRYPOINT dotnet test

# Imagem usada para a fase de execução (executar)
FROM microsoft/dotnet:2.1-aspnetcore-runtime AS final
WORKDIR /app
COPY --from=build /src/app.dll /app
ENTRYPOINT dotnet app.dll
EXPOSE 80 443
```

### Angular

```dockerfile
# Imagem usada para a fase de construção (restaurar, compilar, testar)
FROM agileek/ionic-framework AS build
WORKDIR /src
EXPOSE 80 443

# Argumentos necessários para a compilação do projeto
ARG RUN_TEST=false

# Instalar e configurar ferramentas


# Restaurar os pacotes


# Compilar o projeto

# Imagem usada para a fase de execução (executar)
FROM nginx AS final
WORKDIR /app
EXPOSE 80 443
```

