# Dockerfile

## Objetivo

Criar o arquivo [Dockerfile](https://docs.docker.com/engine/reference/builder/), o qual é responsável por compilar, testar, executar o projeto.

## Padrões

- Deve expor os resultados dos testes no caminho `/TestResults`


## Exemplos

### JAVA

```dockerfile
# Imagem usada para a fase de construção (restaurar, compilar, testar)
FROM maven:3.5.4-jdk-8 AS build
WORKDIR /src
EXPOSE 80 443

# Argumentos necessários para a compilação do projeto
ARG SKIP_TEST=true
ARG MAVEN_REGISTRY=http://repo.maven.apache.org/maven2
ARG PROXY_ACTIVE=false
ARG PROXY_PROTOCOL=http
ARG PROXY_HOST=proxy.local
ARG PROXY_PORT=3128
ARG PROXY_USERNAME
ARG PROXY_PASSWORD

ARG JAVA_OPTS
ENV JAVA_OPTS=$JAVA_OPTS

ARG SONARQUBE_HOST=http://localhost:9000
ENV SONARQUBE_HOST=$SONARQUBE_HOST

# Instalar e configurar ferramentas

echo "<settings><mirrors><mirror><id>REGISTRY</id><name>REGISTRY</name><url>${MAVEN_REGISTRY}</url><mirrorOf>*</mirrorOf></mirror></mirrors><proxies><proxy><id>PROXY</id><active>${PROXY_ACTIVE}</active><protocol>${PROXY_PROTOCOL}</protocol><host>${PROXY_HOST}</host><port>${PROXY_PORT}</port><username>${PROXY_USERNAME}</username><password>${PROXY_PASSWORD}</password><nonProxyHosts></nonProxyHosts></proxy></proxies><pluginGroups><pluginGroup>org.sonarsource.scanner.maven</pluginGroup></pluginGroups><profiles><profile><id>sonar</id><activation><activeByDefault>true</activeByDefault></activation><properties><sonar.host.url>${SONARQUBE_HOST}</sonar.host.url></properties></profile></profiles></settings>" > settings.xml

# Restaurar os pacotes
COPY pom.xml .
RUN mvn package -Dmaven.test.skip=true -Dspring-boot.repackage.skip=true -s settings.xml

# Compilar o projeto
COPY . .
RUN mvn package -Dmaven.test.skip.exec=$SKIP_TEST -s settings.xml
RUN cp target/*.jar ./app.jar

# Executar os testes
ENTRYPOINT mvn sonar:sonar -Dsonar.host.url=$SONARQUBE_HOST

# Imagem usada para a fase de execução (executar)
FROM openjdk:8-jre AS final
WORKDIR /app
COPY --from=build /src/app.jar /app
ENTRYPOINT exec java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar app.jar
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
ARG NUGET_REGISTRY=https://api.nuget.org/v3/index.json
ARG HTTP_PROXY

# Instalar e configurar ferramentas

RUN echo "<config><add key="http_proxy" value="$HTTP_PROXY" /></config>" > NuGet.Config

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
ARG NPM_REGISTRY=https://registry.npmjs.org/
ARG HTTP_PROXY

# Instalar e configurar ferramentas
RUN npm config set registry ${NPM_REGISTRY}

# Restaurar os pacotes


# Compilar o projeto

# Imagem usada para a fase de execução (executar)
FROM nginx AS final
WORKDIR /app
EXPOSE 80 443
```

