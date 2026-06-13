# ⚙️ SFMC Provisioning Hub (WSProxy & SSJS)

## 📌 O Contexto de Negócio (Cenário)
A **Nexus Retail** (cenário fictício) é uma rede de e-commerce em processo de expansão acelerada para 5 novos países. A cada nova operação regional, o time de CRM enfrentava um gargalo operacional crítico: o setup inicial do Salesforce Marketing Cloud exigia a criação manual de uma complexa árvore de pastas (Data Folders), Data Extensions e SQL Queries. 

Esse processo manual consumia **várias horas mensais** do time de engenharia e apresentava um alto risco de erro humano e inconsistência de nomenclatura, comprometendo a integridade das automações globais.

## 🎯 A Solução
Para resolver este gargalo, desenvolvi o **SFMC Provisioning Hub**: uma CloudPage interativa de autoatendimento (Self-Service Wizard) que orquestra a criação automatizada, padronizada e em lote de *assets* via API SOAP.

O resultado? Uma redução drástica no SLA de setup de infraestrutura: de horas para poucos segundos, garantindo 100% de precisão nas nomenclaturas e relacionamentos das bases de dados.

> **Nota de Arquitetura:** No ambiente real de produção do Marketing Cloud, este projeto opera como um único e massivo bloco de código dentro de uma CloudPage (por restrições de arquitetura da plataforma). Para fins de portfólio e legibilidade, o código neste repositório foi **modularizado**, separando as camadas de Interface (Front-end) e Processamento (Back-end/SSJS).

---

## 🛠️ Stack Tecnológico e Decisões de Engenharia

### 1. Back-end & Integração: `SSJS` e `WSProxy`
* **Performance:** O núcleo da aplicação foi construído 100% em Server-Side JavaScript (SSJS). A comunicação com as tabelas do SFMC não utiliza AMPScript ou funções Core padrão. Em vez disso, implementei o **WSProxy** — um wrapper nativo que interage diretamente com a SOAP API.
* **Por que WSProxy?** É significativamente mais rápido, consome menos recursos de processamento no servidor da Salesforce e permite operações assíncronas de CRUD em lote, essenciais para a geração em massa de pastas e Data Extensions.

### 2. Front-end: `HTML5` e `Vanilla JS`
* **Zero Overhead:** A interface foi construída sem frameworks pesados (como React ou Vue). O uso de Vanilla JS e CSS inline garante renderização instantânea da CloudPage e evita bloqueios de segurança institucionais comuns ao carregar bibliotecas via CDNs externas.

---

## 🔐 Segurança e Tratamento de Exceções

### 1. Autenticação por Cofre Invisível
Nenhuma credencial ou senha é mantida no código (*hardcoded*). A aplicação utiliza a função `Platform.Function.ContentBlockByKey()` para acessar um bloco de texto encriptado no Content Builder que valida as chaves de segurança da operação.

### 2. Modo Simulação (Dry-Run Feature)
Implementação de uma trava de segurança essencial na interface. Quando ativada, o back-end processa e formata todos os payloads, mapeia a hierarquia de pastas (Retrieve), mas ignora as chamadas de escrita (CreateItem). Ele retorna um ID simulado para a cadeia, permitindo testes exaustivos sem poluir a base de dados de produção.

---

## ⚠️ Engineering Gotchas (Lições Aprendidas)

**O "Bug do Stringify" no Motor Jint do SFMC**
Durante a construção do serializador de Payload, documentei um bug nativo e conhecido do compilador Jint (o motor por trás do SSJS do Marketing Cloud). Ao utilizar a função `Stringify()` em arrays complexos derivados de requisições do WSProxy, o motor do SFMC frequentemente trava, retornando um "Erro 500" fatal. 
* **A Resolução:** Contornei a limitação reescrevendo a etapa de serialização dos dados do *Dropdown* de forma manual no back-end (via concatenação de strings customizadas), tornando a aplicação totalmente resiliente contra as quedas globais do motor Jint.
