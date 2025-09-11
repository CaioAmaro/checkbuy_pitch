# =========================================================================
# ESTÁGIO 1: O "CONSTRUTOR" (Builder)
# Imagem com Maven e o JDK 21 (estamos usando a imagem oficial do Eclipse Temurin)
# =========================================================================
FROM maven:3.9.6-eclipse-temurin-21 AS build

# Define o diretório de trabalho dentro do container
WORKDIR /app

# Copia primeiro o pom.xml para aproveitar o cache do Docker
COPY pom.xml .

# Baixa as dependências do projeto
RUN mvn dependency:go-offline

# Agora copia todo o resto do código-fonte do seu projeto
COPY src ./src

# Executa o comando para "empacotar" a aplicação, gerando o .jar
# O -DskipTests pula a execução dos testes para acelerar o deploy
RUN mvn clean package -DskipTests


# =========================================================================
# ESTÁGIO 2: A IMAGEM FINAL (Final Image)
# Agora, usamos uma imagem base super leve, que contém apenas o Java 21
# para RODAR a aplicação (JRE), não para desenvolver.
# =========================================================================
FROM eclipse-temurin:21-jre-alpine

# --- INÍCIO DA SEÇÃO PARA INSTALAR O FIREFOX ---
# Mudar para o usuário root TEMPORARIAMENTE para poder instalar pacotes
USER root

# Instala o Firefox e as fontes necessárias para renderizar páginas corretamente
RUN apk update && \
    apk add --no-cache \
        firefox \
        ttf-freefont \
        font-noto-cjk && \
    # Limpa o cache para manter a imagem final o menor possível
    rm -rf /var/cache/apk/*

# Cria um usuário sem privilégios para rodar a aplicação (boa prática de segurança)
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
# Muda para o novo usuário para o resto dos comandos
USER appuser
# --- FIM DA SEÇÃO PARA INSTALAR O FIREFOX ---


# Define o diretório de trabalho
WORKDIR /app

# Copia APENAS o arquivo .jar que foi gerado no Estágio 1 (build)
# para dentro da nossa imagem final.
COPY --from=build /app/target/*.jar app.jar

# Expõe a porta que sua aplicação Spring Boot usa (o padrão é 8080)
EXPOSE 8080

# O comando final que será executado quando o container iniciar.
# Ele simplesmente roda o .jar da sua aplicação.
ENTRYPOINT ["java", "-jar", "app.jar"]