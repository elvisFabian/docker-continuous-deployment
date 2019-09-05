# NET

```dockerfile

FROM mcr.microsoft.com/dotnet/core/sdk:2.2.202 as base

ARG HTTP_PROXY
ARG HTTPS_PROXY
ARG TimeZone="America/Cuiaba"
ARG SONAR_SCANNER_NUGET_VERSION="4.6.2"
ARG SONAR_SCANNER_BIN_VERSION="3.3.0.1492"
ARG SONAR_SCANNER_NETCORE_VERSION="netcoreapp2.1"
ARG REPORTGENERATOR_NUGET_VERSION="4.2.12"
ARG NUGET_SOURCE_EXTERNO="https://api.nuget.org/v3/index.json"
ARG NUGET_SOURCE_INTERNO="http://nuget.tjmt.jus.br/repository/nuget/"

ENV RESULT_PATH="/TestResults/result/vsTest/"
ENV COVERAGE_PATH="/TestResults/codecoverage/"
ENV COVERAGE_REPORT_PATH="/TestResults/codecoverage/Report/"
ENV SONAR_HOST_URL="http://sonarqube.tjmt.jus.br"
ENV SONAR_LOGIN="c29b8801c173a4d9605a5eba61a069272b80dc7c"
ENV HTTP_PROXY=$HTTP_PROXY
ENV HTTPS_PROXY=$HTTPS_PROXY
#----------------------------------------------------------------------------------------------#

#--------------------------------------Instalar o java-----------------------------------------#
#Necessário para o sonarqube
#https://community.sonarsource.com/t/sonarscanner-fails-with-error-permission-denied/1897
#https://community.sonarsource.com/t/c-sonarscanner-net-core-version-permission-denied-error-in-docker/8354
RUN apt-get update && apt-get install -y openjdk-8-jre
#----------------------------------------------------------------------------------------------#

#--------------------------------------Instalando ferramentas globalmente----------------------#
#https://www.nuget.org/packages/dotnet-sonarscanner/
#https://www.nuget.org/packages/dotnet-reportgenerator-globaltool/
RUN dotnet tool install --global dotnet-sonarscanner --version ${SONAR_SCANNER_NUGET_VERSION} && \
    dotnet tool install --global dotnet-reportgenerator-globaltool --version ${REPORTGENERATOR_NUGET_VERSION}
ENV PATH "$PATH:/root/.dotnet/tools/"
RUN chmod +x /root/.dotnet/tools/.store/dotnet-sonarscanner/${SONAR_SCANNER_NUGET_VERSION}/dotnet-sonarscanner/${SONAR_SCANNER_NUGET_VERSION}/tools/${SONAR_SCANNER_NETCORE_VERSION}/any/sonar-scanner-${SONAR_SCANNER_BIN_VERSION}/bin/sonar-scanner
#----------------------------------------------------------------------------------------------#

#--------------------------------------Configura o TimeZone------------------------------------#
RUN ln -snf /usr/share/zoneinfo/$TimeZone /etc/localtime && echo $TimeZone > /etc/timezone
#----------------------------------------------------------------------------------------------#

#--------------------------------------Copiando arquivos sh------------------------------------#
COPY /entrypoint/ /entrypoint/
COPY /utils/ /utils/

RUN chmod +x /entrypoint/entrypoint.sh \
            /entrypoint/wait-for-it.sh \
            /utils/nuget-config-creator.sh   

RUN /utils/nuget-config-creator.sh || exit 0;
#----------------------------------------------------------------------------------------------#




#-----------------Imagem utilizada como target para rodar os testes de integração - CI---------#
#Necessário informar o SOLUTION_NAME pois na raiz do projeto existe o projeto dcproj (docker-project). Criar como variavel de ambiente, pois precisa ser utilizado no ENTRYPOINT
FROM base as ci
ARG CONFIGURATION="Release"
ENV CONFIGURATION=$CONFIGURATION
ENV SOLUTION_NAME="XXXX.sln"

#====================================================================#
# Se utilizar o copy informando o path dos projetos, será utilizado cache do RESTORE, porém para cada projeto criado, tem que ser adicionado no copy
# Um caso ou outro devido issue https://github.com/moby/moby/issues/15858
COPY ["XXXX.sln", "XXX.sln"]
COPY ["src/path1/projeto1.csproj", "src/path1/"]
COPY ["src/path2/projeto2.csproj", "src/path2/"]
COPY ["test/path3/projeto3.csproj", "test/path3/projeto3/"]
RUN dotnet restore ${SOLUTION_NAME} -v m
COPY . .
#----------------------------------------------------------------------------------------------#


#---------------------------Imagem usada para build---------------------------------------#
FROM ci as build
RUN dotnet build ${SOLUTION_NAME} -c ${CONFIGURATION} -v m


#---------------------------Imagem usada para publicação---------------------------------------#
FROM build as publish
RUN dotnet publish src/path1  -c ${CONFIGURATION} --no-build -o /app/www -v m
RUN dotnet pack src/path2     -c ${CONFIGURATION} --no-build -o /app/package -v m


FROM nexusdocker.tjmt.jus.br/dsa/publicador:latest as release
ARG VERSION=latest
ARG BRANCH
ENV VERSION=${VERSION}
ENV BRANCH=${BRANCH}

COPY . ./source
COPY --from=publish app/www ./www
COPY --from=publish /app/package ./packages/nuget



#-----------------------------Imagem usada para runtime ---------------------------------------#
FROM mcr.microsoft.com/dotnet/core/aspnet:2.2 AS runtime
#---------Argumentos
ARG TimeZone="America/Cuiaba"
ARG Language="pt_BR"
ARG Unicode="UTF-8"

#---------Configura o TimeZone
RUN ln -snf /usr/share/zoneinfo/$TimeZone /etc/localtime && echo $TimeZone > /etc/timezone

#Configurando linguagem
RUN apt-get update && \
    apt-get install -y locales locales-all && \
    locale-gen ${Language}.${Unicode} && \
    update-locale LANG=${Language}.${Unicode}

ENV LANG ${Language}.${Unicode}
ENV LANGUAGE ${Language}

EXPOSE 80 443
COPY --from=publish /app/www /app
WORKDIR /app
ENTRYPOINT dotnet app.dll
```