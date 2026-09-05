<div align="center">

# 🛡️ GG // CYBERLAB

### Blue Team • SOC • DFIR • Incident Response • DevSecOps

Laboratório autoral de cibersegurança desenvolvido para simular uma infraestrutura corporativa segmentada, executar testes controlados, detectar atividades suspeitas e conduzir investigações de segurança.

</div>

---

## 🎯 Sobre o projeto

O **GG CyberLab** é um ambiente prático construído em **VirtualBox**, com foco em operações de segurança defensiva.

O laboratório integra:

- Firewall e segmentação de rede
- Active Directory
- SIEM
- Honeypot
- Aplicação Web vulnerável
- Incident Response
- Digital Forensics
- DevSecOps
- Attack Lab

O objetivo é demonstrar um fluxo completo:

```text
Ataque controlado
        ↓
Geração de eventos
        ↓
Detecção
        ↓
Correlação
        ↓
Resposta a incidente
        ↓
Investigação forense
        ↓
Conclusão
```

---

# 🏗️ Arquitetura

A infraestrutura do **GG CyberLab** foi projetada com segmentação por VLANs, separando usuários, servidores, SOC, DevSecOps, DMZ e o ambiente de ataque.

<p align="center">
  <img src="images/topologia-gg-cyberlab.png" alt="Topologia do GG CyberLab" width="100%">
</p>

<p align="center">
  <i>Topologia lógica do ambiente GG CyberLab.</i>
</p>

---

## 🌐 Segmentação

| VLAN | Segmento | Rede | Função |
|---|---|---|---|
| 10 | USERS | `10.10.10.0/24` | Estações de usuários |
| 20 | SERVERS | `10.10.20.0/24` | Active Directory e servidores |
| 30 | SECURITY / SOC | `10.10.30.0/24` | Wazuh, IRIS e Velociraptor |
| 50 | DMZ | `10.10.50.0/24` | Nginx, Juice Shop e Cowrie |
| 99 | ATTACK LAB | `10.10.99.0/24` | Kali Linux |

---

# 🧰 Stack utilizada

| Tecnologia | Função |
|---|---|
| **OPNsense** | Firewall, roteamento e segmentação |
| **Wazuh** | SIEM / XDR e correlação de eventos |
| **Cowrie** | Honeypot SSH |
| **Nginx** | Reverse Proxy |
| **OWASP Juice Shop** | Aplicação Web vulnerável |
| **Active Directory** | Identidades, autenticação e domínio |
| **DFIR-IRIS** | Gestão de incidentes |
| **Velociraptor** | Investigação DFIR |
| **Gitea** | Repositório Git interno |
| **Trivy** | Vulnerability Scanner |
| **Docker / Podman** | Containers |
| **Kali Linux** | Attack Lab |
| **VirtualBox** | Virtualização |

---

# 🔥 Caso de segurança validado

Um dos cenários realizados no laboratório simulou uma tentativa de autenticação SSH contra o honeypot.

### Origem

```text
Kali Linux
IP: 10.10.99.161
VLAN 99 - ATTACK LAB
```

### Destino

```text
GG_LN05
IP: 10.10.50.10
VLAN 50 - DMZ
```

Fluxo do incidente:

```text
Kali Linux
     │
     ▼
Cowrie Honeypot
     │
     ▼
Logs de segurança
     │
     ▼
Wazuh Agent
     │
     ▼
Wazuh SIEM
     │
     ▼
Alerta customizado
     │
     ▼
DFIR-IRIS
     │
     ▼
Velociraptor
     │
     ▼
Investigação
```

---

# 🍯 Honeypot

O **Cowrie** foi utilizado para simular um serviço SSH exposto e registrar tentativas de acesso.

Eventos capturados:

```text
cowrie.login.failed
cowrie.session.closed
```

O endereço de origem identificado foi:

```text
10.10.99.161
```

---

# 📊 Detecção com Wazuh

Os eventos do Cowrie foram enviados ao **Wazuh** para análise e correlação.

Foi criada uma regra customizada:

```text
Rule ID: 100200

HONEYPOT: Tentativa de login SSH falhou
IP: 10.10.99.161
```

Também foi validada a regra:

```text
Rule ID: 100204

HONEYPOT: Sessao SSH encerrada
```

Com isso, o fluxo abaixo foi comprovado:

```text
Attack Lab → Honeypot → Wazuh Agent → Wazuh Manager → Alert
```

---

# 🚨 Incident Response

O alerta foi registrado no **DFIR-IRIS**.

### Caso

**Tentativa de acesso SSH detectada no Honeypot**

Foram registrados:

- IOC
- Asset
- Evidência
- Tasks
- Timeline
- Classificação
- Severidade
- Investigação DFIR

### IOC

```text
10.10.99.161
Type: ip-src
```

### Ativo relacionado

```text
GG_LN05-HONEYPOT
10.10.50.10
```

---

# 🔎 Investigação DFIR

A investigação foi realizada utilizando o **Velociraptor**.

Foram coletados e analisados artefatos relacionados a:

- Processos
- Conexões de rede
- Portas em escuta
- Tarefas agendadas
- Usuários privilegiados
- Histórico de login
- Eventos SSH

### Artefatos utilizados

```text
Linux.Sys.Pslist
Linux.Network.Netstat
Linux.Sys.Crontab
Linux.Sys.LastUserLogin
Linux.Syslog.SSHLogin
Linux.Users.RootUsers
```

### Resultado

Durante a análise não foram identificados:

- processos maliciosos;
- contas privilegiadas suspeitas;
- persistências anômalas;
- conexões incompatíveis com o ambiente.

Os serviços identificados estavam de acordo com a arquitetura do laboratório:

```text
Cowrie
Nginx
Docker
SSH
Velociraptor Client
Juice Shop
```

> **Conclusão:** nenhum indicador de comprometimento foi identificado nos artefatos coletados durante a investigação.

A atividade observada permaneceu restrita ao ambiente controlado do honeypot.

---

# 🌐 Web Security

O laboratório possui uma aplicação Web vulnerável destinada exclusivamente a testes controlados.

```text
Attack Lab
    │
    ▼
  Nginx
    │
    ▼
OWASP Juice Shop
```

O **Juice Shop** é executado em container e publicado através do **Nginx**.

Esse ambiente permite estudar:

- Segurança Web
- Logs HTTP
- Reverse Proxy
- Detecção
- Monitoramento
- Resposta a eventos

---

# 🪟 Active Directory

O ambiente também simula uma infraestrutura corporativa Microsoft.

Domínio:

```text
gglab.corp
```

O Active Directory fornece:

- usuários corporativos;
- grupos;
- autenticação;
- DNS;
- servidores Windows;
- estações ingressadas no domínio.

Eventos de segurança dos servidores Windows podem ser enviados ao Wazuh para correlação.

---

# 🔐 DevSecOps

O CyberLab também contempla um ambiente de DevSecOps.

```text
Developer
    │
    ▼
  Gitea
    │
    ▼
 Pipeline
    │
    ▼
  Trivy
    │
    ▼
Vulnerability Scan
```

O objetivo é incorporar segurança ao ciclo de desenvolvimento através de:

- versionamento;
- pipelines;
- vulnerability scanning;
- containers;
- análise automatizada.

---

# 🧠 Competências demonstradas

### Blue Team
`SIEM` `Log Analysis` `Detection Engineering` `Threat Detection`

### DFIR
`Incident Response` `Digital Forensics` `IOC Analysis` `Evidence Collection`

### Infrastructure
`OPNsense` `VLAN` `Active Directory` `DNS` `Linux` `Windows Server`

### Security
`Honeypot` `Web Security` `Network Security` `Segmentation`

### DevSecOps
`Git` `Gitea` `Trivy` `Containers` `Pipelines`

---

# 📚 Documentação

A documentação será separada por área:

```text
docs/
│
├── arquitetura.md
├── wazuh.md
├── honeypot.md
├── web-security.md
├── active-directory.md
├── incident-response.md
├── velociraptor.md
└── devsecops.md
```

---

# ⚠️ Disclaimer

Este laboratório foi desenvolvido exclusivamente para:

- estudo;
- pesquisa;
- aprendizado;
- desenvolvimento profissional;
- testes autorizados.

Todos os testes ofensivos foram executados somente contra ativos pertencentes ao próprio ambiente **GG CyberLab**.

---

<div align="center">

## 👤 Geovanni Andrade

**Cybersecurity • Blue Team • SOC • DFIR • Infrastructure**

[GitHub](https://github.com/geovanniandrade)

---

### GG // CYBERLAB

**Detect • Investigate • Respond • Improve**

</div>
