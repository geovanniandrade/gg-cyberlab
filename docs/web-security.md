# 🌐 Web Security — Nginx + OWASP Juice Shop

## Visão Geral

O **GG CyberLab** possui uma camada Web localizada na **DMZ**, utilizada para simular uma aplicação exposta e permitir testes controlados de segurança.

O ambiente Web é composto principalmente por:

- **Nginx**
- **OWASP Juice Shop**
- **Docker**
- **Wazuh Agent**
- **Velociraptor Client**

O objetivo é reproduzir uma arquitetura em que uma aplicação Web vulnerável fique protegida por uma camada de proxy reverso e possa ser monitorada pelas ferramentas do SOC.

---

# 🏗️ Arquitetura Web

O ambiente Web está concentrado no servidor:

```text
Hostname: GG_LN05
IP: 10.10.50.10
VLAN: 50 - DMZ
Sistema Operacional: Ubuntu Server
```

Fluxo:

```text
Attack Lab / Cliente
        │
        ▼
10.10.50.10:80
        │
        ▼
      Nginx
        │
        ▼
127.0.0.1:3000
        │
        ▼
OWASP Juice Shop
```

---

# 🌐 DMZ

Rede:

```text
10.10.50.0/24
```

Segmento:

```text
VLAN 50 - DMZ
```

A DMZ concentra os serviços que podem receber conexões de outras redes do laboratório.

No GG_LN05 são executados:

```text
Nginx
OWASP Juice Shop
Cowrie Honeypot
Docker
Wazuh Agent
Velociraptor Client
```

---

# 🌐 Nginx

O **Nginx** foi utilizado como camada de entrada para a aplicação Web.

Função principal:

```text
Reverse Proxy
```

Porta publicada:

```text
TCP/80
```

Durante os testes, o serviço foi observado em:

```text
0.0.0.0:80
```

Processo associado:

```text
nginx
```

---

# 🔄 Reverse Proxy

A função do Nginx é receber as requisições HTTP enviadas ao GG_LN05 e encaminhá-las para o Juice Shop.

Fluxo:

```text
Cliente
   │
   ▼
http://10.10.50.10
   │
   ▼
Nginx :80
   │
   ▼
Juice Shop :3000
```

Com essa arquitetura, o usuário não precisa acessar diretamente a porta do container.

---

# 🧪 OWASP Juice Shop

O **OWASP Juice Shop** é uma aplicação propositalmente vulnerável utilizada exclusivamente para estudos e testes autorizados.

Ela permite praticar conceitos relacionados a:

- vulnerabilidades Web;
- análise de requisições;
- autenticação;
- segurança de aplicação;
- logs;
- monitoramento;
- resposta a incidentes.

---

# 🐳 Execução em Container

O Juice Shop é executado em container.

Durante a validação do ambiente foi observado que o serviço permanecia associado apenas ao endereço:

```text
127.0.0.1:3000
```

Processo responsável pela publicação:

```text
docker-proxy
```

Isso significa que a aplicação não estava diretamente exposta para outras VLANs através da porta 3000.

---

# 🔐 Restrição da Porta 3000

A configuração utilizada permite separar a aplicação da camada de exposição.

```text
Juice Shop
127.0.0.1:3000
```

A porta:

```text
3000/tcp
```

permanece local ao servidor.

O acesso externo ocorre através do:

```text
Nginx
TCP/80
```

Fluxo:

```text
Rede
  │
  ▼
10.10.50.10:80
  │
  ▼
Nginx
  │
  ▼
127.0.0.1:3000
  │
  ▼
Juice Shop
```

---

# ✅ Validação de Exposição

Durante os testes realizados a partir da máquina Kali, foi validado que:

```text
TCP/80
OPEN
```

enquanto:

```text
TCP/3000
CLOSED / não exposta externamente
```

Esse comportamento confirma que o Juice Shop é acessado externamente somente através do Nginx.

---

# 🐉 Attack Lab

A máquina utilizada para testes pertence à:

```text
VLAN 99 - ATTACK LAB
```

Rede:

```text
10.10.99.0/24
```

Sistema:

```text
Kali Linux
```

O Attack Lab permite validar a exposição dos serviços sem utilizar sistemas externos ao laboratório.

---

# 🔥 Fluxo de Teste

Exemplo do fluxo utilizado:

```text
Kali Linux
10.10.99.161
      │
      ▼
HTTP Request
      │
      ▼
GG_LN05
10.10.50.10:80
      │
      ▼
Nginx
      │
      ▼
Juice Shop
127.0.0.1:3000
```

---

# 🛡️ Separação entre Web e Honeypot

O GG_LN05 executa simultaneamente dois serviços destinados a testes de segurança:

```text
TCP/22
→ Cowrie Honeypot

TCP/80
→ Nginx
→ OWASP Juice Shop
```

Essa configuração permite gerar diferentes tipos de telemetria no mesmo ativo.

---

# 📊 Monitoramento

O Wazuh Agent instalado no GG_LN05 pode ser utilizado para centralizar eventos relacionados ao servidor.

Fluxo lógico:

```text
Nginx / Sistema Operacional
          │
          ▼
      Wazuh Agent
          │
          ▼
      Wazuh Manager
          │
          ▼
      SOC / Analysis
```

No cenário documentado do CyberLab, a correlação customizada mais detalhada foi validada principalmente com os eventos do Cowrie.

---

# 🔎 DFIR com Velociraptor

O GG_LN05 também possui o **Velociraptor Client**.

Durante a investigação foram analisadas conexões de rede e portas em escuta.

Entre os resultados observados:

```text
0.0.0.0:80
Process: nginx
```

e:

```text
127.0.0.1:3000
Process: docker-proxy
```

Esses resultados eram compatíveis com a arquitetura esperada do ambiente Web.

---

# 🧠 Interpretação

A arquitetura permite que:

- o Nginx seja a camada de exposição;
- o Juice Shop permaneça acessível apenas localmente;
- a aplicação seja executada em container;
- o tráfego Web passe por um ponto central;
- o host seja monitorado pelo SOC;
- o endpoint possa ser investigado via Velociraptor.

---

# 🔐 Benefícios da Arquitetura

Mesmo sendo um laboratório, essa configuração reproduz conceitos comuns em ambientes reais.

### Reverse Proxy

```text
Cliente → Nginx → Aplicação
```

### Isolamento de Serviço

```text
Juice Shop
127.0.0.1:3000
```

### Segmentação

```text
Web Server
VLAN 50 - DMZ
```

### Monitoramento

```text
GG_LN05 → Wazuh
```

### DFIR

```text
GG_LN05 → Velociraptor
```

---

# 🚨 Possível Fluxo de Incident Response

Um evento Web relevante pode seguir o fluxo:

```text
Web Activity
    │
    ▼
Nginx / Application Logs
    │
    ▼
Wazuh
    │
    ▼
Security Alert
    │
    ▼
DFIR-IRIS
    │
    ▼
Velociraptor
    │
    ▼
Investigation
```

---

# 🧪 Ambiente de Testes

O OWASP Juice Shop foi utilizado exclusivamente dentro do ambiente controlado do CyberLab.

O objetivo é permitir o estudo de segurança de aplicações sem expor sistemas de terceiros.

Os testes são realizados somente contra:

```text
GG_LN05
10.10.50.10
```

pertencente ao próprio laboratório.

---

# 📸 Evidências

As evidências desta camada podem ser organizadas em:

```text
images/
└── web-security/
    ├── nginx-status.png
    ├── juice-shop.png
    ├── docker-container.png
    ├── port-80.png
    ├── port-3000.png
    └── velociraptor-netstat.png
```

Essas imagens podem demonstrar:

- Nginx em execução;
- Juice Shop acessível;
- container ativo;
- porta 80 publicada;
- porta 3000 restrita;
- resultados de rede coletados pelo Velociraptor.

---

# 📌 Resumo

A camada Web do **GG CyberLab** utiliza:

```text
VLAN 50 - DMZ
       │
       ▼
    GG_LN05
       │
   ┌───┴────┐
   │        │
   ▼        ▼
Nginx     Cowrie
   │
   ▼
Juice Shop
```

O resultado é um ambiente controlado para estudos de:

```text
Web Security
Reverse Proxy
Containers
Network Segmentation
Monitoring
Incident Response
DFIR
```

---

[⬅️ Voltar para Honeypot](honeypot.md)

[🏠 Voltar para o README](../README.md)
