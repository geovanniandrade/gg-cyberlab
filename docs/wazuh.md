# 📊 Wazuh — SIEM / XDR

## Visão Geral

O **Wazuh** é a principal plataforma de monitoramento e detecção utilizada no **GG CyberLab**.

Ele atua como o núcleo do SOC, recebendo eventos provenientes de diferentes ativos do laboratório e permitindo:

- coleta centralizada de logs;
- monitoramento de endpoints;
- análise de eventos;
- correlação;
- criação de regras customizadas;
- geração de alertas;
- investigação de eventos;
- integração com o processo de resposta a incidentes.

---

# 🏗️ Arquitetura do Wazuh

O Wazuh foi instalado no servidor:

```text
Hostname: GG_LN01
VLAN: 30 - SOC
IP: 10.10.30.10
Sistema Operacional: Ubuntu Server
```

A instalação utilizada no laboratório é do tipo:

```text
Wazuh All-in-One
```

Os principais componentes utilizados são:

- Wazuh Manager;
- Wazuh Indexer;
- Wazuh Dashboard;
- Filebeat.

Fluxo:

```text
Endpoints
    │
    ▼
Wazuh Agents
    │
    ▼
Wazuh Manager
    │
    ▼
Wazuh Indexer
    │
    ▼
Wazuh Dashboard
```

---

# 🖥️ Servidor do SOC

Servidor:

```text
GG_LN01
```

Endereço:

```text
10.10.30.10
```

Segmento:

```text
VLAN 30 - SOC
```

A VLAN SOC foi criada para concentrar as ferramentas responsáveis por:

- monitoramento;
- correlação;
- investigação;
- resposta a incidentes;
- análise forense.

---

# ⚙️ Serviços do Wazuh

Durante a implementação foram utilizados os seguintes componentes:

```text
wazuh-manager
wazuh-indexer
wazuh-dashboard
filebeat
```

A arquitetura lógica funciona da seguinte maneira:

```text
Log / Telemetria
       │
       ▼
Wazuh Agent
       │
       ▼
Wazuh Manager
       │
       ▼
Decoder / Rules
       │
       ▼
Wazuh Indexer
       │
       ▼
Wazuh Dashboard
```

---

# 🛰️ Agentes

Os agentes permitem que o Wazuh receba informações dos ativos monitorados.

No CyberLab foram utilizados agentes em sistemas Linux e Windows.

Exemplos de ativos integrados:

```text
GG_LN05-HONEYPOT
OPNsense
GG_WS01
GG_WS02
GG_WS03
```

### 📸 Evidência — Agentes monitorados

<p align="center">
  <img src="../images/wazuh/wazuh-agents.png" alt="Agentes monitorados pelo Wazuh" width="100%">
</p>

<p align="center">
  <i>Visão dos agentes integrados ao Wazuh no ambiente GG CyberLab.</i>
</p>

---

## GG_LN05 — Honeypot / Web

Servidor:

```text
GG_LN05
```

Endereço:

```text
10.10.50.10
```

Funções:

```text
Cowrie Honeypot
Nginx
OWASP Juice Shop
Docker
```

O agente associado ao servidor apareceu no Wazuh como:

```text
GG-LN05-HONEYPOT
```

Durante os testes, o agente enviou ao Wazuh os eventos gerados pelo Cowrie.

---

# 🍯 Integração Cowrie → Wazuh

O Cowrie gera logs estruturados em formato JSON.

Durante o cenário de teste foram observados eventos como:

```text
cowrie.login.failed
cowrie.session.closed
```

Fluxo:

```text
Kali Linux
10.10.99.161
      │
      ▼
Cowrie Honeypot
10.10.50.10
      │
      ▼
Cowrie JSON Logs
      │
      ▼
Wazuh Agent
      │
      ▼
Wazuh Manager
      │
      ▼
Custom Rules
      │
      ▼
Security Alert
```

---

# 🔥 Cenário de Detecção

A máquina Kali localizada na VLAN 99 foi utilizada para gerar uma tentativa controlada de autenticação SSH.

### Origem

```text
IP: 10.10.99.161
VLAN: 99
Segmento: ATTACK LAB
```

### Destino

```text
IP: 10.10.50.10
Hostname: GG_LN05
VLAN: 50
Segmento: DMZ
```

Serviço de destino:

```text
TCP/22
Cowrie Honeypot
```

---

# 🧠 Regra Customizada 100200

Foi utilizada uma regra customizada do Wazuh para identificar tentativas de autenticação SSH registradas pelo Cowrie.

Identificador:

```text
Rule ID: 100200
```

Alerta observado:

```text
HONEYPOT: Tentativa de login SSH falhou
```

Endereço IP de origem identificado:

```text
10.10.99.161
```

Fluxo:

```text
cowrie.login.failed
        │
        ▼
Wazuh
        │
        ▼
Rule 100200
        │
        ▼
Security Alert
```

### 📸 Evidência — Rule 100200

<p align="center">
  <img src="../images/wazuh/wazuh-rule-100200.png" alt="Wazuh Rule 100200" width="100%">
</p>

<p align="center">
  <i>Detecção de tentativa de autenticação SSH no Cowrie correlacionada pela regra customizada 100200 do Wazuh.</i>
</p>

---

# 🔎 Regra Customizada 100204

Também foi observada a regra:

```text
Rule ID: 100204
```

Responsável pela identificação do encerramento da sessão registrada pelo honeypot.

Exemplo:

```text
HONEYPOT: Sessao SSH encerrada
IP: 10.10.99.161
```

Evento associado:

```text
cowrie.session.closed
```

---

# 🚨 Threat Hunting

Os eventos foram analisados através da interface de **Threat Hunting** do Wazuh.

Durante os testes, os alertas permitiram identificar:

- agente responsável pelo evento;
- endereço IP de origem;
- regra disparada;
- mensagem da detecção;
- informações relacionadas à sessão;
- timestamp do evento.

Exemplo de correlação:

```text
Agent
GG-LN05-HONEYPOT

Source IP
10.10.99.161

Destination
10.10.50.10

Rule
100200

Event
cowrie.login.failed
```

---

# 🔗 Correlação do Evento

O cenário permitiu validar a seguinte cadeia:

```text
ATTACK LAB
    │
    ▼
Kali Linux
10.10.99.161
    │
    ▼
SSH Attempt
    │
    ▼
Cowrie Honeypot
GG_LN05
10.10.50.10
    │
    ▼
JSON Log
    │
    ▼
Wazuh Agent
    │
    ▼
Wazuh Manager
    │
    ▼
Rule 100200
    │
    ▼
Security Alert
```

---

# 🚨 Integração com Incident Response

Após a identificação do alerta, o evento foi utilizado para abertura de um caso na plataforma **DFIR-IRIS**.

Fluxo:

```text
Wazuh
   │
   ▼
Security Alert
   │
   ▼
DFIR-IRIS
   │
   ▼
Incident Case
   │
   ├── IOC
   ├── Asset
   ├── Evidence
   ├── Tasks
   └── Timeline
         │
         ▼
   Velociraptor
         │
         ▼
  DFIR Investigation
```

---

# 📋 Informações transferidas para o IRIS

### Regra

```text
100200
```

### IP de origem

```text
10.10.99.161
```

### Ativo relacionado

```text
GG_LN05-HONEYPOT
10.10.50.10
```

### Tipo de atividade

```text
Tentativa de autenticação SSH
```

---

# 🪟 Monitoramento do Active Directory

O ambiente também possui servidores Windows integrados à arquitetura de monitoramento.

Principal controlador de domínio:

```text
GG_WS01
10.10.20.10
```

Domínio:

```text
gglab.corp
```

A integração permite trabalhar com eventos relacionados a:

- autenticação;
- falhas de login;
- contas;
- grupos;
- alterações administrativas;
- eventos de segurança do Windows.

> O cenário de detecção detalhado neste documento corresponde principalmente ao fluxo Cowrie → Wazuh, utilizado como caso prático de validação do SOC.

---

# 🔎 Papel do Wazuh no CyberLab

Dentro do projeto, o Wazuh representa a camada de:

```text
Security Monitoring
        +
SIEM
        +
Detection Engineering
        +
Threat Hunting
        +
Endpoint Monitoring
```

Ele conecta a telemetria dos ativos ao processo de resposta a incidentes.

---

# 🛡️ Fluxo Completo de Segurança

```text
           TELEMETRIA
               │
               ▼
        ┌─────────────┐
        │    WAZUH    │
        │    SIEM     │
        └──────┬──────┘
               │
         Detection Rules
               │
               ▼
             Alert
               │
               ▼
        ┌─────────────┐
        │    IRIS     │
        │ Incident IR │
        └──────┬──────┘
               │
               ▼
        ┌─────────────┐
        │Velociraptor │
        │    DFIR     │
        └─────────────┘
```

---

# ✅ Resultado

O cenário permitiu comprovar o funcionamento do pipeline de detecção:

```text
Kali
  ↓
Cowrie
  ↓
Wazuh Agent
  ↓
Wazuh Manager
  ↓
Custom Rule
  ↓
Alert
  ↓
IRIS
  ↓
Velociraptor
```

O Wazuh identificou os eventos gerados pelo honeypot e os transformou em alertas utilizados durante o processo de resposta a incidentes.

---

# 📸 Evidências Utilizadas

As evidências deste documento estão armazenadas em:

```text
images/
└── wazuh/
    ├── wazuh-agents.png
    └── wazuh-rule-100200.png
```

Essas evidências demonstram:

- integração dos agentes;
- monitoramento centralizado;
- detecção do Cowrie;
- regra customizada 100200;
- eventos associados à Rule 100204;
- funcionamento do Threat Hunting.

---

# 📌 Resumo

O Wazuh funciona como o centro de monitoramento do **GG CyberLab**.

```text
Logs
  ↓
Wazuh
  ↓
Detection
  ↓
Alert
  ↓
IRIS
  ↓
Velociraptor
  ↓
Investigation
```

Com isso, o CyberLab demonstra um fluxo completo de **coleta, detecção, correlação, resposta e investigação**.

---

[⬅️ Voltar para Arquitetura](arquitetura.md)

[🏠 Voltar para o README](../README.md)
