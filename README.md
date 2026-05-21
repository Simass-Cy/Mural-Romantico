# Mural Romantico API

API backend do projeto Mural Romantico, um SaaS para criacao e gerenciamento de murais digitais romanticos.

## Stack

- Java 17
- Spring Boot
- Maven Wrapper
- Spring Web
- Spring Security
- Spring Data JPA
- Flyway
- PostgreSQL

## Requisitos

- Java 17 instalado
- PostgreSQL para execucao local quando a persistencia estiver configurada
- Variaveis de ambiente configuradas localmente

## Configuracao local

1. Copie `.env.example` para `.env`.
2. Preencha o `.env` com valores locais reais.
3. Nunca versione `.env`, segredos, tokens, certificados, dumps de banco ou dados reais de usuarios.

O arquivo versionado `src/main/resources/application.properties` deve permanecer sem segredos reais. Configuracoes sensiveis devem vir de variaveis de ambiente, arquivos locais ignorados pelo Git ou secrets do ambiente de deploy.

## Comandos

Execute a partir da raiz do projeto:

```powershell
.\mvnw.cmd spring-boot:run
.\mvnw.cmd test
.\mvnw.cmd clean package
```

## Status do projeto

Este repositorio esta sendo preparado para o primeiro commit publico. A implementacao da API deve seguir o escopo documentado em `docs/`.

## Seguranca

Antes de cada commit, revise o diff e confirme que nenhum dado critico foi incluido. Em especial, nao comite:

- `.env` real ou arquivos `.env.*` locais
- senhas, tokens, JWT secrets ou credenciais de provedores
- chaves privadas, certificados ou keystores
- dumps de banco de dados
- dados reais de usuarios
- URLs assinadas ou payloads privados
