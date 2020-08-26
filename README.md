# Docker

Estudos sobre o funcionamento e gestão de containers utilizando Docker, seguindo a orientação do *Getting Started*, 
utilizado no [Docker Desktop](https://www.docker.com/products/docker-desktop).

## Containers
Containers são simplesmente outro processo que foi isolado dos demais processos na máquina *host*. Potencializando os [namespaces](#namespaces) e [cgroups](#cgroups) do kernel, recursos que já existem no Linux há muito tempo e que o Docker tornou acessíveis e fáceis de usar.

### <a name="namespaces"></a> Namespaces
Namespaces é uma *feature* que permite criar e lidar com diversos contextos em um mesmo sistema, vendo propriedades
globais diferentes e isoladas em cada contexto. Um exemplo prático é a possibilidade de criar um contexto (ou ambiente) de rede isolado do ambiente físico, onde existirão interfaces de rede física que não são visíveis no contexto do sistema. Estas interfaces terão endereços físicos e lógicos diferentes do sistema e todo tráfego, regras de firewall, existentes nesse novo contexto e não são vistos por nenhum outro contexto, incluindo a máquina *host*.

[Referência](https://medium.com/@lets00/namespace-14c4e64d0559) completa.

### <a name="cgroups"></a> Cgroups (Grupos de Controle)
Cgroups é um recurso que limita, contabiliza e isola o uso de recursos de uma coleção de processos.

## Rodando a aplicação *Getting Started*
```sh
# Roda um container baseado na imagem passada
# docker run <imagem>
docker run -d -p 80:80 docker/getting-started
```

### Flags do `docker run`
- `-d` - executa o container em modo background (*detached mode*)
- `-p 80:80` - faz o mapeamento da porta do host para a porta do container *host:container*
- `docker/getting-started` - imagem utilizada

> As flags podem ser combinadas para diminuir o comando: <br>
> `docker run -dp 80:80 docker/getting-started

### Imagem do container
O filesystem utilizando na execução de um container é provida pela imagem do Container. Esta imagem precisa conter tudo para rodar uma aplicação - todas as dependências, configurações, scripts, arquivos binários, etc. A imagem também contém outras configurações para o container, como variáveis de ambiente, comandos padrões de execução e demais metadatas.

## Comandos
- `docker ps` - Visualiza todos os containers
- `docker stop <container-id>` - Interrompe a execução de um container
- `docker rm -f <container-id>` - Remove o container
- `docker login -u <username>` - Efetua login local com um usuário do DockerHub
- `docker tag nome usuario/image` - Adicionar uma tag padrão para a imagem local criada através do Dockerfile
- `docker push usuario/image` - Envia uma imagem local criada através do Dockerfile para o DockerHub
- `docker volume inspect <volume>` - Inspeciona os dados do volume (data, labels, caminho, nome, escopo)
- `docker logs -f <container-id>` - Visualiza os logs do container

## Volumes
Volumes são utilizados para compartilhar informações entre o host e o container, além de também ser utilizado caso seja necessário compartilhar informações/pastas/db-files entre containers.

```sh
# Cria um novo volume com o nome informado
# docker volume create <nome-do-volume>
docker volume create todo-db

# Adiciona o volume criado ao rodar o container
# Ex.: docker run -v <nome-do-volume>:<caminho-container> <imagem>
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```

## Pastas vinculadas (Bind Mounts)
Diferentemente dos volumes, que são nomeados e não precisamos nos preocupar *onde* a informação é armazenada, as pastas vinculadas controlamos exatamente o ponto de vinculação no host local. Podemos utilizar para persistir dados mas é mais utilizado para prover mais informações para os containers.

Um exemplo prático é no caso de uma aplicação, podemos manter o código original em uma máquina e a aplicação rodando dentro de um container, podendo validar as modificações do código imediatamente.

Para comparação entre os Volumes e os Bind Mounts.

|                        | Volumes                         | Bind Mounts            |
| ---------------------- |:-------------------------------:| ----------------------:|
| Caminho local          | Docker define                   | Usuário define         |
| Exemplo usando -v      | meu-volume:/usr/local/data      | /path/:/usr/local/data |
| Preenche o volume com <br>dados do container | Sim       | Não                    |
| Suporta Volume de Drivers | Sim                          | Não                    |

### Iniciando um container em *Dev-Mode*
```sh
docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/all" \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```
- `-dp 3000:3000` - Roda em modo detached e cria um mapeamento de porta
- `-w /app` - Define o diretório de trabalho, ou o diretório onde serão executados comandos
- `-v "$(pwd):/app"` - Vincula a pasta de trabalho atual ao diretório `/app` do container
- `node:12-alpine` - É a imagem a ser utilizada
- `sh -c "yarn install && yarn run dev"` - O comando que será utilizado. Iniciando um shell usando `sh` (a imagem alpine não tem `bash`) e rodando `yarn install` para resolver todas as dependências do projeto e finalmente `yarn run dev`, que deve estar definido no `package.json` do projeto. Levando em consideração que o projeto do exemplo estaria rodando uma aplicação JS/TS/Node...

A utilização das pastas vinculadas (*Bind Mounts*) é muito usada em setups de desenvolvimento. A grande vantagem é que a máquina local não necessita todas as ferramentas de build e ambientes instaladas. Com um simples comando `docker run` o ambiente de desenvolvimento está iniciado e pronto para ser utilizado.

## Dockerfile
Dockerfile são simplesmente scripts descritos com instruções que são usadas para a criação de uma imagem.
O Docker file não pode conter extensão.

```sh
# Compila as informações do Dockerfile
# docker build -t usuario/imagem:tag .
docker build -t andersonkoester/getting-started:1.0
```
Este comando é usado para gerar uma nova imagem de container baseada no Dockerfile, todas as dependências são baixadas para que o builder 
