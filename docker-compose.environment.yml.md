Responsável por conter as configurações (variáveis de ambiente, labels, etc) e imagem a ser utilizada para a publicação em um ambiente específico.
> Pode-se haver múltiplos arquivos que será utilizado pela ferramenta de automação para criar/publicar no ambiente específico.
>
> Exemplo:
> - `docker-compose.dev.yml`
> - `docker-compose.qa.yml`
> - `docker-compose.stage.yml`
> - `docker-compose.prod.yml`