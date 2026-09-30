# Hands-on: Evolução e Otimização de Microcontainers

**Ambiente de Execução:** Github Codespace (https://github.com/codespaces)
**Objetivo:** Demonstrar na prática a redução do footprint de memória e o aumento da segurança (redução da superfície de ataque) em microsserviços através da evolução de imagens Docker.

## Parte 1: Preparação do Ambiente e da Aplicação

1. Crie o diretório do projeto e acesse-o:
```bash
mkdir puc-arquitetura
cd puc-arquitetura/

```

2. Crie o arquivo principal da aplicação:

```bash
nano app.js

```

3. Cole o código JavaScript abaixo e salve o arquivo:

```javascript
const express = require('express')
const app = express()

app.get('/', (req, res) => res.send('PUC Arquitetura \n Hello World! 0 container esta rodando.'))

app.listen(3000, () => console.log('Server ready na porta 3000'))

```

4. Inicialize o projeto e instale o framework Express:

```bash
npm init -y
npm install express

```

5. Execute a aplicação:

```bash
node app.js

```

## Parte 2: O Padrão Oficial (Gargalo de Tamanho)

1. Crie o arquivo Docker padrão:

```bash
nano Dockerfile

```

2. Insira as instruções da imagem oficial do Node.js:

```dockerfile
FROM node:24
WORKDIR /usr/src/app
COPY package*.json app.js ./
RUN npm install
EXPOSE 3000
CMD ["node", "app.js"]

```

3. Construa a imagem:

```bash
docker build -t node-image-official .

```

4. Execute o container:

```bash
docker run -d --name node-oficial-container -p 3000:3000 node-image-official

```

## Parte 3: Otimização de Tamanho com Alpine

O Alpine Linux é uma distribuição voltada para segurança e tamanho, utilizando musl libc no lugar da tradicional glibc.

1. Crie o arquivo Docker para o Alpine:

```bash
nano Dockerfile-alpine

```

2. Insira as instruções de construção:

```dockerfile
FROM alpine
RUN apk update && apk upgrade
RUN apk add nodejs npm
WORKDIR /usr/src/app
COPY package*.json app.js ./
RUN npm install
EXPOSE 3000
CMD ["node", "app.js"]

```

3. Construa a imagem:

```bash
docker build -t node-image-alpine -f Dockerfile-alpine .

```

4. Execute o container:

```bash
docker run -d --name node-alpine-container -p 3000:3000 node-image-alpine

```

## Parte 4: Otimização com Alpine e Multi-stage Build

Para evitar que ferramentas de compilação e gerenciadores de pacotes (como o npm) cheguem ao ambiente de produção, dividimos a construção em dois estágios.

1. Crie o arquivo Docker com múltiplos estágios:

```bash
nano Dockerfile-alpine-multi

```

2. Insira as instruções:

```dockerfile
# Estágio 1: Builder (Prepara as dependências)
FROM alpine AS builder
RUN apk add --no-cache nodejs npm
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install --only=production
COPY app.js ./

# Estágio 2: Produção (Imagem limpa sem npm)
FROM alpine
RUN apk add --no-cache nodejs
WORKDIR /usr/src/app
COPY --from=builder /usr/src/app ./
EXPOSE 3000
CMD ["node", "app.js"]

```

3. Construa a imagem intermediária:

```bash
docker build -t node-image-alpine-multi -f Dockerfile-alpine-multi .

```

4. Execute o container:

```bash
docker run -d --name node-alpine-container-multi -p 3000:3000 node-image-alpine-multi

```

## Parte 5: Foco Extremo em Segurança com Distroless

As imagens Distroless (mantidas pelo Google) não contêm gerenciadores de pacotes, shells ou utilitários de sistema operacional (como ls, grep ou bash). Elas carregam estritamente o runtime da aplicação.

1. Crie o arquivo Docker para a versão Distroless:

```bash
nano Dockerfile-distroless

```

2. Insira as instruções utilizando o Debian oficial para o build e o Distroless para a produção:

```dockerfile
# Estágio 1: Builder (Ambiente completo)
FROM node:24 AS builder
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install --only=production
COPY app.js ./

# Estágio 2: Produção (Imagem segura, sem sistema operacional visível)
FROM gcr.io/distroless/nodejs24-debian12
WORKDIR /usr/src/app
COPY --from=builder /usr/src/app ./
EXPOSE 3000
CMD ["app.js"]

```

3. Construa a imagem de alta segurança:

```bash
docker build -t node-image-distroless -f Dockerfile-distroless .

```

4. Execute o container em segundo plano:

```bash
docker run -d --name distroless-app -p 3000:3000 node-image-distroless

```

5. Teste de Intrusão (Superfície de Ataque): Tente abrir um terminal dentro do container em execução para validar o isolamento de segurança. O comando abaixo deve falhar, provando que o invasor não tem ferramentas de sistema operacional disponíveis:

```bash
docker exec -it distroless-app sh

```

## Parte 6: Comparação de Resultados

1. Liste as imagens geradas para comparar os tamanhos (footprints):

```bash
docker images

```

```

```
