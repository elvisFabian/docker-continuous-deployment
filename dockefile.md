# Dockefile

## Objetivo

Criar o arquivo [Dockerfile](https://docs.docker.com/engine/reference/builder/), o qual é responsável por compilar, testar, executar o projeto.


## Exemplos

### JAVA

```dockerfile
# Imagem usada para a fase de construção (restaurar, compilar, testar)
FROM maven:3.5.4-jdk-8 AS build

# Argumentos necessários para a compilação do projeto


# Imagem usada para a fase de execução (executar)
FROM openjdk:8-jre AS final
WORKDIR /app
EXPOSE 80
EXPOSE 443
```

### .NET

```dockerfile
# Imagem usada para a fase de construção (restaurar, compilar, testar)
FROM microsoft/dotnet:2.1-sdk AS build

# Argumentos necessários para a compilação do projeto


# Imagem usada para a fase de execução (executar)
FROM microsoft/dotnet:2.1-aspnetcore-runtime AS final
WORKDIR /app
EXPOSE 80
EXPOSE 443
```

### Angular

```dockerfile
# Imagem usada para a fase de construção (restaurar, compilar, testar)
FROM agileek/ionic-framework AS build

# Argumentos necessários para a compilação do projeto


# Imagem usada para a fase de execução (executar)
FROM nginx AS final
WORKDIR /app
EXPOSE 80
EXPOSE 443
```

