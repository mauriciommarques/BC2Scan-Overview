# BC2Scan - Visão de Arquitetura

Este repositório apresenta a arquitetura e o funcionamento do sistema BC2Scan.  
O código-fonte permanece privado.

---

## 📍 TL;DR

BC2Scan é um sistema de indexação de blockchain projetado para processar dados de forma assíncrona e escalável, garantindo que o usuário sempre receba informações atualizadas e consistentes, mesmo sob carga.

---

## 📍 Sobre

BC2Scan é um sistema de indexação e consulta assíncrona para a rede BC2, projetado para escalar a leitura de dados sem depender de reprocessamentos completos a cada consulta.

O sistema separa claramente entrada, processamento e entrega, garantindo que o usuário nunca receba dados inconsistentes ou antigos.

---

## 🧠 Filosofia de Design

O BC2Scan foi construído com base na separação de responsabilidades:

- Entrada (API)  
- Orquestração (Filas)  
- Processamento (Workers)  
- Entrega (Banco + Front-End)  

Isso garante:

- Escalabilidade  
- Consistência  
- Clareza arquitetural  

---

## 🧩 Arquitetura

O sistema é dividido em múltiplos serviços independentes:

- **BC2Scan.Api** → Entrada e validação de requisições  
- **BC2Scan.Queues** → Roteamento e controle de fluxo (carteiro)  
- **BC2Scan.Processos** → Execução pesada (fullscan no node)  
- **BC2Scan.Core** → Serviços compartilhados  
- **BC2Scan.DeviceApi** → Endpoints específicos de dispositivos  
- **BC2Scan.SEO** → Indexação e geração de sitemap  
- **WebApp** → Interface do usuário  

---

## 🔄 Fluxo Completo do Sistema

### 1. Usuário / Front-End (WebApp)

O usuário acessa a interface e realiza uma consulta:

- Ação: clicar em **Consultar**  
- Requisição: `POST /scan`  

---

### 2. BC2Scan.Api

A API recebe e valida a requisição:

- Valida input  
- Publica mensagem no RabbitMQ  
  - Exchange: `miuta.scan`  

---

### 3. BC2Scan.Queues (Worker - Carteiro)

Responsável por orquestrar o fluxo:

- Consome `scan.request`  
- Valida estrutura JSON  
- Roteia para próxima etapa (`scan.result`)  
- Mantém estado inicial **NEW**  
- Não acessa banco de dados  

---

### 4. BC2Scan.Processos (Worker - Executor)

Executa o processamento pesado:

- Atualiza status para **PROCESSING**  
- Executa fullscan no node BC2  
- Gera JSON consolidado  
- Atualiza banco de dados (linha única)  
- Finaliza com status **READY**  

---

### 5. Banco de Dados (MySQL)

Modelo baseado em consistência:

- Cada endereço possui uma única linha  
- Armazena JSON completo consolidado  

Controle de estado:

- **PROCESSING** → ainda em execução  
- **READY** → pronto para consumo  

**Regra crítica:**  
O sistema nunca entrega dados antigos.

---

### 6. Front-End / Usuário (Watcher)

A interface acompanha o status da consulta:

- Endpoint: `/status`  

Fluxo:

- Se **READY**:
  - Busca JSON consolidado  
  - Atualiza a interface  

- Pode aplicar timeout (ex: 15s)

**Garantia:**  
O usuário só vê dados atualizados e consistentes.

---

## ⚙️ Princípios do Sistema

- Processamento assíncrono por design  
- Separação clara de responsabilidades  
- Nenhum worker executa mais do que deveria  
- Banco de dados como fonte única de verdade  
- Zero entrega de dados obsoletos  

---

## 🚀 Operação

O deploy e operação do sistema são desacoplados do código principal.

Scripts de build, deploy e manutenção estão disponíveis em repositório separado:

- **BC2-SCRIPTS**

Essa separação garante:

- Código limpo  
- Operação controlada  
- Deploy reproduzível  

---

## 🏁 Status do Projeto

BC2Scan encontra-se em estado estável e funcional.

A partir deste ponto:

- Novas mudanças são incrementais  
- Cada commit representa evolução clara  
- A arquitetura base está consolidada  

---

## 📌 Observação Final

BC2Scan não foi projetado apenas para funcionar —  
foi projetado para manter consistência sob carga e crescimento.
