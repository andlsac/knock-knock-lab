# knock-knock-lab
Laboratório prático de auditoria de segurança para demonstrar vulnerabilidades de força bruta. Ambiente configurado com Kali/Metasploitable 2, executando ataques com Medusa e propondo medidas de mitigação para os serviços testados. #Pentest #EthicalHacking #DIO



    🚫 🚨 AVISO LEGAL 🚨 🚫

    ❗ Disclaimer de Uso:
    › Apenas para fins educacionais e de pesquisa
    › Use por sua conta e risco
    › Não use para atividades ilegais
    › Responsabilidade é exclusivamente do usuário




# Desafio de Projeto DIO: Laboratório de Pentest com Kali Linux e Medusa

![Licença](https://img.shields.io/badge/license-MIT-blue.svg)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-557C94?style=for-the-badge&logo=kali-linux&logoColor=white)
![VMware Fusion](https://img.shields.io/badge/VMware_Fusion-607078?style=for-the-badge&logo=vmware&logoColor=white)

## Sumário

- [Visão Geral do Projeto](#visão-geral-do-projeto)
- [Objetivos de Aprendizagem](#objetivos-de-aprendizagem)
- [Fase 1: Configuração do Ambiente (Setup do Laboratório)](#fase-1-configuração-do-ambiente-setup-do-laboratório)
- [Fase 2: Reconhecimento (Information Gathering)](#fase-2-reconhecimento-information-gathering)
- [Fase 3: Execução dos Ataques (Exploitation)](#fase-3-execução-dos-ataques-exploitation)
  - [Cenário 1: Força Bruta em Serviço FTP](#cenário-1-força-bruta-em-serviço-ftp)
  - [Cenário 2: Força Bruta em Formulário Web (DVWA)](#cenário-2-força-bruta-em-formulário-web-dvwa)
  - [Cenário 3: Password Spraying em Serviço SMB](#cenário-3-password-spraying-em-serviço-smb)
- [Fase 4: Análise de Riscos e Recomendações (Mitigação)](#fase-4-análise-de-riscos-e-recomendações-mitigação)
- [Conclusão e Aprendizados](#conclusão-e-aprendizados)
- [Licença](#licença)
- [Autor](#autor)

---

## Visão Geral do Projeto

Este repositório documenta a execução do Desafio de Projeto da **[Digital Innovation One (DIO)](https://www.dio.me/)**, focado na simulação de ataques de força bruta em um ambiente de laboratório controlado. O objetivo é demonstrar a aplicação prática de técnicas de pentest, utilizando o Kali Linux e a ferramenta Medusa contra alvos vulneráveis, como o Metasploitable 2 e a aplicação web DVWA.

Todo o processo, desde a configuração do ambiente até a proposição de medidas de segurança, está detalhado neste documento.

--------------------------------------------------------------------------------------------------------------------------------------------------

## Objetivos de Aprendizagem

Ao final deste desafio, os seguintes objetivos foram alcançados:

-   **Compreensão Prática:** Entendimento aprofundado de como ataques de força bruta são executados contra diferentes protocolos (FTP, HTTP, SMB).
-   **Uso de Ferramentas:** Utilização proficiente do Kali Linux e do Medusa para realizar auditorias de segurança de forma ética.
-   **Documentação Técnica:** Habilidade de documentar um processo de pentest de forma clara, estruturada e reprodutível.
-   **Análise de Vulnerabilidades:** Reconhecimento de fraquezas comuns relacionadas a senhas e autenticação.
-   **Proposição de Soluções:** Capacidade de recomendar contramedidas e boas práticas de segurança para mitigar os riscos identificados.

--------------------------------------------------------------------------------------------------------------------------------------------------

## Fase 1: Configuração do Ambiente (Setup do Laboratório)

Para garantir um ambiente seguro e isolado, toda a simulação foi realizada em máquinas virtuais.

### 1.1. Software Utilizado
- **Virtualizador:**  VMWare e UTM `versão X.X.X`
- **Máquina de Ataque:** Kali Linux `versão 2025.X`
- **Máquina Alvo:** Metasploitable 2

### 1.2. Configuração de Rede
Ambas as máquinas virtuais foram configuradas para usar uma rede do tipo **"Rede Apenas de Hospedeiro (Host-Only)"**. Isso cria uma rede privada entre a máquina hospedeira e as VMs, isolando completamente o tráfego do laboratório da rede externa.

- **Endereço IP - Kali Linux:** `xxx.xxx.xxx.xxx`  - **Endereço IP - Metasploitable 2:** `192.168.1.9` ### 1.3. Verificação de Conectividade
Após a configuração, a conectividade entre as máquinas foi validada com o comando `ping`.

**Comando (executado no Kali):**
```bash
ping -c 4 192.168.1.9
```
**Evidência:**
![Verificação de Ping](images/ping_check.png)

--------------------------------------------------------------------------------------------------------------------------------------------------

## Fase 2: Reconhecimento(Information Gathering) e Análise com Ferramentas Automatizadas

Antes de qualquer ataque, um reconhecimento foi realizado para identificar os serviços e portas abertas no alvo. A ferramenta `nmap` foi utilizada para este fim.

**Comando (executado no Kali):**
```bash
nmap -sV -p- 192.168.1.9
```
**Parâmetros do Comando:**
- `-sV`: Tenta determinar a versão dos serviços em execução nas portas abertas.
- `-p-`: Escaneia todas as 65535 portas TCP.

**Resultados:**
O scan revelou diversas portas abertas, incluindo:
- `Porta 21/tcp`: Serviço **FTP** (vsftpd 2.3.4)
- `Porta 22/tcp`: Serviço **SSH** OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
- `Porta 80/tcp`: Serviço **HTTP** (Apache httpd 2.2.8 ((Ubuntu) DAV/2)
- `Porta 445/tcp`: Serviço **netbios-ssn** (amba smbd 3.X - 4.X (workgroup: WORKGROUP))
Esses serviços foram selecionados como alvos para a próxima fase.

**Evidência:**
![Resultado do Nmap](images/nmap_scan.png)

----------------------------------------------------------------------------------------------------------------------------------

## Análise de Vulnerabilidades com Ferramentas Automatizadas

### 2.1 Scanner de Servidor Web (Nikto)
**Objetivo:** Identificar problemas de configuração e vulnerabilidades conhecidas no servidor.

**Output completo disponível em:** `./logs/nikto_scan.txt`

**Principais Vulnerabilidades Detectadas:**

| Vulnerabilidade | Risco | Evidência |
|----------------|------------|-----------|
| Servidor Apache desatualizado (2.2.8) | Alto | Versão EOL identificada |
| Arquivo phpinfo.php exposto | Alto | Informações do sistema acessíveis |
| Diretórios sensíveis acessíveis (/doc/, /test/) | Médio | Listagem de diretórios ativa |
| Método TRACE ativo | Médio | Vulnerável a Cross-Site Tracing |
| phpMyAdmin acessível sem restrições | Alto | Painel administrativo exposto |

**Nota Técnica:**
- **Servidor:** Apache/2.2.8 (Ubuntu) + PHP/5.2.4
- **Problemas de Configuração:** Cabeçalhos de segurança ausentes (X-Frame-Options, X-Content-Type)
- **Arquivos Expostos:** phpinfo.php, phpMyAdmin, diretórios do sistema

**Recomendação Imediata:** Atualizar servidor web e restringir acesso a arquivos sensíveis.

## 2.2 Scanner de Aplicação Web (Wapiti)

**Objetivo:** Analisar vulnerabilidades específicas na aplicação DVWA.

**Output completo disponível em:** `./logs/scan_wapiti.txt`

**Principais Vulnerabilidades Detectadas:**

| Categoria | Vulnerabilidade | Risco | Evidência |
|-----------|----------------|------------|-----------|
| **Content Security Policy** | CSP não configurado | Médio | Falta cabeçalho Content-Security-Policy |
| **HTTP Headers** | X-Frame-Options ausente | Médio | Permite clickjacking |
| **HTTP Headers** | X-XSS-Protection ausente | Médio | Sem proteção contra XSS |
| **HTTP Headers** | X-Content-Type-Options ausente | Médio | Permite MIME sniffing |
| **HTTP Headers** | Strict-Transport-Security ausente | Médio | Sem forçar HTTPS |
| **Cookies** | HttpOnly flag não configurada (PHPSESSID) | Médio | Cookie acessível via JavaScript |
| **Cookies** | HttpOnly flag não configurada (security) | Médio | Cookie acessível via JavaScript |
| **Cookies** | Secure flag não configurada (PHPSESSID) | Médio | Cookie transmitido em texto claro |
| **Cookies** | Secure flag não configurada (security) | Médio | Cookie transmitido em texto claro |

### Resumo das Vulnerabilidades por Categoria

| Categoria | Quantidade | Status |
|-----------|------------|---------|
| HTTP Security Headers | 4 vulnerabilidades | Crítico |
| Cookie Security | 4 vulnerabilidades | Crítico |
| Content Security Policy | 1 vulnerabilidade |  Médio |
| SQL Injection | 0 vulnerabilidades | Seguro |
| XSS | 0 vulnerabilidades | Seguro |
| Path Traversal | 0 vulnerabilidades |  Seguro |

### Impacto Geral

**Configurações de segurança inadequadas:**
- Ataques cross-site (XSS)
- Clickjacking
- Exposição de dados sensíveis
- Session hijacking

### Recomendações

1. **Implementar cabeçalhos de segurança HTTP**
2. **Configurar flags de segurança em cookies**
3. **Adotar Content Security Policy**
4. **Forçar uso de HTTPS**
5. **Proteger contra clickjacking**
6. **Bloquear MIME sniffing**
7. **Isolar cookies de scripts client-side**

--------------------------------------------------------------------------------------------------------------------------------------------------

## Fase 3: Execução dos Ataques (Exploitation)

Com os alvos identificados, a ferramenta **Medusa** foi utilizada para executar os ataques de força bruta.

### Cenário 1: Força Bruta em Serviço FTP

- **Objetivo:** Obter acesso não autorizado ao serviço FTP (porta 21) no Metasploitable 2.
- **Wordlists Utilizadas:**
  - `usuarios.txt` (contendo `root`, `admin`, `msfadmin`)
  - `senhas.txt` (contendo `toor`, `admin`, `msfadmin`, `password`)

**Comando (executado no Kali):**
```bash
medusa -h 192.168.1.9 -U usuarios.txt -P senhas.txt -M ftp
```
**Parâmetros do Comando:**
`-h` 192.168.1.9: O parâmetro -h (de host) especifica o alvo do ataque, que é o endereço IP 192.168.1.9.
`-U` usuarios.txt: O parâmetro -U (de User file) indica o arquivo que contém a lista de usuários a serem testados.
`-P` senhas.txt: O parâmetro -P (de Password file) indica o arquivo que contém a lista de senhas a serem testadas para cada usuário.
`-M` ftp: O parâmetro -M (de Module) define qual serviço (módulo) será atacado. Neste caso, foi o ftp.)

**Funcionamento:**
O Medusa testa automaticamente todas as combinações de usuários e senhas dos arquivos contra o serviço FTP do servidor 192.168.1.9.

**Resultados:**
O ataque foi bem-sucedido, revelando a credencial válida: **`msfadmin` / `msfadmin`**.

**Evidência:**
![Sucesso no Ataque FTP](images/ftpimagem.png)

--------------------------------------------------------------------------------------------------------------------------------------------------
### Cenário 2: Força Bruta em Formulário Web (DVWA)

- **Objetivo:** Comprometer o formulário de login da página "Brute Force" do Damn Vulnerable Web Application (DVWA), rodando no Metasploitable 2 e configurado com a segurança no nível `Low`.

**Comando (executado no Kali):**
```bash
hydra -L usuarios.txt -P senhas.txt 192.168.1.9 http-post-form \
"/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:F=Login failed"
```
**Parâmetros do Comando:**
- `-L usuarios.txt`: Arquivo contendo lista de usuários para teste.
- `-P senhas.txt`: Arquivo contendo lista de senhas para teste.
- `192.168.1.9`: IP do servidor/alvo do ataque
- `http-post-form`: Módulo para ataque em formulários web via POST
- `-m FORM:"..."`: Define a URL e os parâmetros do formulário, usando `^USER^` e `^PASS^` como placeholders.

**Estrutura do Formulário:**
- `/dvwa/login.php`: URL do formulário.
- `username=^USER^&password=^PASS^&Login=Login`: Parâmetros do formulário, usando `^USER^` e `^PASS^` como placeholders.
- `F=Login failed`: Mensagem de erro esperada.

**Funcionamento:**
O Hydra testa automaticamente todas as combinações de usuários e senhas nos arquivos, substituindo ^USER^ e ^PASS^ a cada tentativa, e para quando encontrar credenciais válidas (quando a mensagem "Login failed" não aparece).

**Resultados:**
[80][http-post-form] host: 192.168.1.9   login: admin   password: password

A senha para o usuário `admin` foi descoberta: **`password`**.

**Evidência:**
![Sucesso no Ataque DVWA](images/dvwa_success.png)

__________________________________________________________________________________________________________________________________________________

## Cenário 3: Password Spraying em Serviço SMB

- **Objetivo:** Executar um ataque de *Password Spraying* contra o serviço SMB (porta 445), testando uma única senha fraca contra múltiplos usuários.
- **Senhas Testadas:** `password` e `msfadmin`

**Comando (executado no Kali):**
```bash
hydra -L usuarios.txt -p msfadmin 192.168.1.9 smb -V
```
**Parâmetros do Comando:**
- `-L usuarios.txt`: Arquivo contendo lista de usuários para teste
- `-p msfadmin`: Usa uma senha fixa (msfadmin) para todos os usuários
- `192.168.1.9`: IP do servidor/alvo do ataque
- `smb`: Módulo para ataque ao protocolo SMB (Server Message Block - compartilhamento de arquivos Windows)
- `-V`: Modo verbose - mostra cada tentativa em tempo real

**Funcionamento:****
O Hydra testa cada usuário do arquivo usuarios.txt com a senha fixa msfadmin no serviço SMB do alvo, mostrando todas as tentativas na tela devido ao -V.

**Diferença do anterior:**
    Antes: Ataque a formulário web com múltiplas senhas
    Agora: Ataque a serviço SMB com senha fixa e múltiplos usuários

**Resultados:**
O ataque identificou que o usuário **`msfadmin`** possuía a senha `msfadmin`, permitindo o acesso aos compartilhamentos de rede.

**Evidência:**
![Sucesso no Ataque SMB](images/smbimagem.png)

--------------------------------------------------------------------------------------------------------------------------------------------------


## Fase 4: Análise de Riscos e Recomendações (Mitigação)

*Os ataques demonstraram que senhas fracas representam um risco crítico para a segurança dos serviços. As seguintes contramedidas são recomendadas:*

| Risco Identificado | Medida de Mitigação | Por que é Importante? |
| :------------------| :------------------ | :-------------------- |
| **Senhas Fracas e Padrão** | Implementar uma **Política de Senhas Fortes** (comprimento, complexidade, histórico). | Dificulta exponencialmente a adivinhação de senhas por ferramentas automatizadas. |
| **Tentativas de Login Ilimitadas** | Configurar o **Bloqueio de Contas (Account Lockout)** após X tentativas falhas. | Impede que um atacante possa testar milhões de senhas em um curto período, tornando o ataque inviável. |
| **Automação de Ataques Web** | Utilizar **CAPTCHA** ou reCAPTCHA em formulários de login. | Adiciona uma camada que exige interação humana, quebrando a maioria dos scripts de força bruta. |
| **Comprometimento de Credenciais** | Habilitar a **Autenticação de Múltiplos Fatores (MFA)**. | **A medida mais eficaz.** Mesmo que a senha seja roubada, o atacante não terá o segundo fator (token, biometria, etc). |
| **Falta de Visibilidade** | Implementar **Monitoramento e Alertas** para múltiplas falhas de login. | Permite que a equipe de segurança detecte e responda a um ataque em andamento antes que ele seja bem-sucedido. |

--------------------------------------------------------------------------------------------------------------------------------------------------

## Conclusão

O laboratório evidenciou vulnerabilidades críticas que comprometem a segurança dos sistemas. Os pontos mais urgentes para correção são:

### Pontos Críticos a Serem Corrigidos Imediatamente:

1. **Credenciais Padrão e Fracas**
   - Remover senhas padrão como "msfadmin" e "password"
   - Implementar política de senhas complexas com mínimo de 12 caracteres

2. **Proteção Contra Força Bruta**
   - Configurar bloqueio automático de contas após 5 tentativas falhas
   - Implementar delays progressivos entre tentativas de login

3. **Serviços Desatualizados**
   - Atualizar Apache 2.2.8 para versão suportada
   - Atualizar PHP 5.2.4 para versão atual
   - Correção do vsftpd 2.3.4 vulnerável

4. **Configurações de Segurança Web**
   - Implementar cabeçalhos de segurança (CSP, X-Frame-Options)
   - Configurar flags de segurança em cookies (HttpOnly, Secure)
   - Remover arquivos sensíveis como phpinfo.php

5. **Falta de Autenticação Multi-Fator**
   - Implementar MFA para acesso administrativo
   - Adicionar camada extra de proteção em logins críticos

---

## Licença

Este projeto está licenciado sob a Licença MIT. Consulte o arquivo `LICENSE` para mais detalhes.

---

## Autor

**André Luís Alves Campos**

-------------------------------------------------------------------------

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/andlsac)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/andlsac)
[![Instagram](https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white)](https://instagram.com/andlsac)

----

# ENGLISH Version

### knock-knock-lab

Practical security auditing lab to demonstrate brute-force vulnerabilities. Environment configured with Kali/Metasploitable 2, executing attacks with Medusa and proposing mitigation measures for the tested services. #Pentest #EthicalHacking #DIO

# DIO Project Challenge: Pentest Lab with Kali Linux and Medusa

🚫 🚨 LEGAL WARNING 🚨 🚫

    ❗ Usage Disclaimer:
    › Educational and research purposes only
    › Use at your own risk
    › Do not use for illegal activities
    › User bears all responsibility

![alt text](https://img.shields.io/badge/license-MIT-blue.svg)

  

![alt text](https://img.shields.io/badge/Kali_Linux-557C94?style=for-the-badge&logo=kali-linux&logoColor=white)

  

![alt text](https://img.shields.io/badge/VMware_Fusion-607078?style=for-the-badge&logo=vmware&logoColor=white)

## Table of Contents

- [Project Overview](https://www.google.com/url?sa=E&q=#project-overview)
    
- [Learning Objectives](https://www.google.com/url?sa=E&q=#learning-objectives)
    
- [Phase 1: Environment Setup (Lab Configuration)](https://www.google.com/url?sa=E&q=#phase-1-environment-setup-lab-configuration)
    
- [Phase 2: Reconnaissance (Information Gathering)](https://www.google.com/url?sa=E&q=#phase-2-reconnaissance-information-gathering)
    
- [Phase 3: Attack Execution (Exploitation)](https://www.google.com/url?sa=E&q=#phase-3-attack-execution-exploitation)
    
    - [Scenario 1: Brute-Force on FTP Service](https://www.google.com/url?sa=E&q=#scenario-1-brute-force-on-ftp-service)
        
    - [Scenario 2: Brute-Force on Web Form (DVWA)](https://www.google.com/url?sa=E&q=#scenario-2-brute-force-on-web-form-dvwa)
        
    - [Scenario 3: Password Spraying on SMB Service](https://www.google.com/url?sa=E&q=#scenario-3-password-spraying-on-smb-service)
        
- [Phase 4: Risk Analysis and Recommendations (Mitigation)](https://www.google.com/url?sa=E&q=#phase-4-risk-analysis-and-recommendations-mitigation)
    
- [Conclusion and Learnings](https://www.google.com/url?sa=E&q=#conclusion-and-learnings)
    
- [License](https://www.google.com/url?sa=E&q=#license)
    
- [Author](https://www.google.com/url?sa=E&q=#author)
    

---

## Project Overview

This repository documents the execution of the **[Digital Innovation One (DIO)](https://www.google.com/url?sa=E&q=https%3A%2F%2Fwww.dio.me%2F)** Project Challenge, focused on simulating brute-force attacks in a controlled lab environment. The goal is to demonstrate the practical application of pentesting techniques, using Kali Linux and the Medusa tool against vulnerable targets such as Metasploitable 2 and the DVWA web application.

The entire process, from environment setup to proposing security measures, is detailed in this document.

---

## Learning Objectives

Upon completion of this challenge, the following objectives were achieved:

- **Practical Understanding:** Deep understanding of how brute-force attacks are executed against different protocols (FTP, HTTP, SMB).
    
- **Tool Usage:** Proficient use of Kali Linux and Medusa to perform security audits ethically.
    
- **Technical Documentation:** Ability to document a pentesting process clearly, structured, and reproducibly.
    
- **Vulnerability Analysis:** Recognition of common weaknesses related to passwords and authentication.
    
- **Solution Proposition:** Capacity to recommend countermeasures and security best practices to mitigate identified risks.
    

---

## Phase 1: Environment Setup (Lab Configuration)

To ensure a secure and isolated environment, the entire simulation was performed on virtual machines.

### 1.1. Software Used

- **Virtualizer:** VMWare and UTM version X.X.X
    
- **Attack Machine:** Kali Linux version 2025.X
    
- **Target Machine:** Metasploitable 2
    

### 1.2. Network Configuration

Both virtual machines were configured to use a **"Host-Only Network"**. This creates a private network between the host machine and the VMs, completely isolating the lab traffic from the external network.

- **IP Address - Kali Linux:** xxx.xxx.xxx.xxx - **IP Address - Metasploitable 2:** 192.168.1.9 ### 1.3. Connectivity Verification  
    After configuration, connectivity between the machines was validated with the ping command.
    

**Command (executed on Kali):**

code Bash

downloadcontent_copy

expand_less

    `ping -c 4 192.168.1.9`
  

**Evidence:**  
Here's an image showing the ping check:
![Verificação de Ping](images/ping_check.png)


----
## Phase 2: Reconnaissance (Information Gathering) and Automated Tool Analysis

Before any attack, reconnaissance was performed to identify open services and ports on the target. The `nmap` tool was used for this purpose.

**Command (executed on Kali):**
```bash
nmap -sV -p- 192.168.1.9
```
**Command Parameters:**
- `-sV`: Attempts to determine the version of services running on open ports.
- `-p-`: Scans all 65535 TCP ports.

**Results:**
The scan revealed several open ports, including:
- `Port 21/tcp`: **FTP** service (vsftpd 2.3.4)
- `Port 22/tcp`: **SSH** service OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
- `Port 80/tcp`: **HTTP** service (Apache httpd 2.2.8 ((Ubuntu) DAV/2)
- `Port 445/tcp`: **netbios-ssn** service (amba smbd 3.X - 4.X (workgroup: WORKGROUP))
These services were selected as targets for the next phase.

**Evidence:**
Here's an image showing the Nmap scan results: 
![Resultado do Nmap](images/nmap_scan.png)




----------------------------------------------------------------------------------------------------------------------------------

## Vulnerability Analysis with Automated Tools

### 2.1 Web Server Scanner (Nikto)
**Objective:** Identify configuration issues and known vulnerabilities on the server.

**Full output available in:** `./logs/nikto_scan.txt`

**Key Vulnerabilities Detected:**

| Vulnerability | Risk | Evidence |
|---|---|---|
| Outdated Apache server (2.2.8) | High | EOL version identified |
| Exposed phpinfo.php file | High | System information accessible |
| Accessible sensitive directories (/doc/, /test/) | Medium | Directory listing active |
| TRACE method active | Medium | Vulnerable to Cross-Site Tracing |
| phpMyAdmin accessible without restrictions | High | Administrative panel exposed |

**Technical Note:**
- **Server:** Apache/2.2.8 (Ubuntu) + PHP/5.2.4
- **Configuration Issues:** Missing security headers (X-Frame-Options, X-Content-Type)
- **Exposed Files:** phpinfo.php, phpMyAdmin, system directories

**Immediate Recommendation:** Update web server and restrict access to sensitive files.

## 2.2 Web Application Scanner (Wapiti)

**Objective:** Analyze specific vulnerabilities in the DVWA application.

**Full output available in:** `./logs/scan_wapiti.txt`

**Key Vulnerabilities Detected:**

| Category | Vulnerability | Risk | Evidence |
|---|---|---|---|
| **Content Security Policy** | CSP not configured | Medium | Missing Content-Security-Policy header |
| **HTTP Headers** | X-Frame-Options missing | Medium | Allows clickjacking |
| **HTTP Headers** | X-XSS-Protection missing | Medium | No XSS protection |
| **HTTP Headers** | X-Content-Type-Options missing | Medium | Allows MIME sniffing |
| **HTTP Headers** | Strict-Transport-Security missing | Medium | No HTTPS enforcement |
| **Cookies** | HttpOnly flag not configured (PHPSESSID) | Medium | Cookie accessible via JavaScript |
| **Cookies** | HttpOnly flag not configured (security) | Medium | Cookie accessible via JavaScript |
| **Cookies** | Secure flag not configured (PHPSESSID) | Medium | Cookie transmitted in clear text |
| **Cookies** | Secure flag not configured (security) | Medium | Cookie transmitted in clear text |

### Summary of Vulnerabilities by Category

| Category | Quantity | Status |
|---|---|---|
| HTTP Security Headers | 4 vulnerabilities | Critical |
| Cookie Security | 4 vulnerabilities | Critical |
| Content Security Policy | 1 vulnerability | Medium |
| SQL Injection | 0 vulnerabilities | Secure |
| XSS | 0 vulnerabilities | Secure |
| Path Traversal | 0 vulnerabilities | Secure |

### Overall Impact

**Inadequate security configurations:**
- Cross-site attacks (XSS)
- Clickjacking
- Exposure of sensitive data
- Session hijacking

### Recommendations

1.  **Implement HTTP security headers**
2.  **Configure security flags on cookies**
3.  **Adopt Content Security Policy**
4.  **Enforce HTTPS usage**
5.  **Protect against clickjacking**
6.  **Block MIME sniffing**
7.  **Isolate cookies from client-side scripts**

--------------------------------------------------------------------------------------------------------------------------------------------------

## Phase 3: Attack Execution (Exploitation)

With the targets identified, the **Medusa** tool was used to execute brute-force attacks.

### Scenario 1: Brute-Force on FTP Service

-   **Objective:** Gain unauthorized access to the FTP service (port 21) on Metasploitable 2.
-   **Wordlists Used:**
    -   `usuarios.txt` (containing `root`, `admin`, `msfadmin`)
    -   `senhas.txt` (containing `toor`, `admin`, `msfadmin`, `password`)

**Command (executed on Kali):**
```bash
medusa -h 192.168.1.9 -U usuarios.txt -P senhas.txt -M ftp
```
**Command Parameters:**
`-h` 192.168.1.9: The -h parameter (for host) specifies the target of the attack, which is the IP address 192.168.1.9.
`-U` usuarios.txt: The -U parameter (for User file) indicates the file containing the list of users to be tested.
`-P` senhas.txt: The -P parameter (for Password file) indicates the file containing the list of passwords to be tested for each user.
`-M` ftp: The -M parameter (for Module) defines which service (module) will be attacked. In this case, it was ftp.

**How it works:**
Medusa automatically tests all combinations of users and passwords from the files against the FTP service on server 192.168.1.9.

**Results:**
The attack was successful, revealing the valid credential: **`msfadmin` / `msfadmin`**.

**Evidence:**
Here's an image showing the successful FTP attack: 
![Sucesso no Ataque FTP](images/ftpimagem.png)
---



--------------------------------------------------------------------------------------------------------------------------------------------------
### Scenario 2: Brute-Force on Web Form (DVWA)

-   **Objective:** Compromise the login form of the "Brute Force" page of the Damn Vulnerable Web Application (DVWA), running on Metasploitable 2 and configured with security at `Low` level.

**Command (executed on Kali):**
```bash
hydra -L usuarios.txt -P senhas.txt 192.168.1.9 http-post-form \
"/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:F=Login failed"
```
**Command Parameters:**
- `-L usuarios.txt`: File containing a list of users to test.
- `-P senhas.txt`: File containing a list of passwords to test.
- `192.168.1.9`: IP of the server/target of the attack.
- `http-post-form`: Module for attacking web forms via POST.
- `-m FORM:"..."`: Defines the URL and form parameters, using `^USER^` and `^PASS^` as placeholders.

**Form Structure:**
- `/dvwa/login.php`: Form URL.
- `username=^USER^&password=^PASS^&Login=Login`: Form parameters, using `^USER^` and `^PASS^` as placeholders.
- `F=Login failed`: Expected error message.

**How it works:**
Hydra automatically tests all combinations of users and passwords from the files, substituting ^USER^ and ^PASS^ with each attempt, and stops when valid credentials are found (when the "Login failed" message does not appear).

**Results:**
[80][http-post-form] host: 192.168.1.9 login: admin password: password

The password for user `admin` was discovered: **`password`**.

**Evidence:**
Here's an image showing the successful DVWA attack: 
![Sucesso no Ataque DVWA](images/dvwa_success.png)

-----------



__________________________________________________________________________________________________________________________________________________

## Scenario 3: Password Spraying on SMB Service

-   **Objective:** Execute a *Password Spraying* attack against the SMB service (port 445), testing a single weak password against multiple users.
-   **Passwords Tested:** `password` and `msfadmin`

**Command (executed on Kali):**
```bash
hydra -L usuarios.txt -p msfadmin 192.168.1.9 smb -V
```
**Command Parameters:**
- `-L usuarios.txt`: File containing a list of users to test.
- `-p msfadmin`: Uses a fixed password (msfadmin) for all users.
- `192.168.1.9`: IP of the server/target of the attack.
- `smb`: Module for attacking the SMB protocol (Server Message Block - Windows file sharing).
- `-V`: Verbose mode - shows each attempt in real-time.

**How it works:**
Hydra tests each user from the `usuarios.txt` file with the fixed password `msfadmin` on the target's SMB service, showing all attempts on the screen due to the `-V` flag.

**Difference from previous scenario:**
    Before: Web form attack with multiple passwords.
    Now: SMB service attack with a fixed password and multiple users.

**Results:**
The attack identified that the user **`msfadmin`** had the password `msfadmin`, allowing access to network shares.

**Evidence:**
Here's an an image showing the successful SMB attack: 
![Sucesso no Ataque SMB](images/smbimagem.png)
---


## Phase 4: Risk Analysis and Recommendations (Mitigation)

*The attacks demonstrated that weak passwords pose a critical risk to service security. The following countermeasures are recommended:*

| Identified Risk | Mitigation Measure | Why It's Important? |
| :------------------| :------------------ | :-------------------- |
| **Weak and Default Passwords** | Implement a **Strong Password Policy** (length, complexity, history). | Exponentially makes password guessing by automated tools more difficult. |
| **Unlimited Login Attempts** | Configure **Account Lockout** after X failed attempts. | Prevents an attacker from testing millions of passwords in a short period, making the attack unfeasible. |
| **Web Attack Automation** | Use **CAPTCHA** or reCAPTCHA on login forms. | Adds a layer that requires human interaction, breaking most brute-force scripts. |
| **Credential Compromise** | Enable **Multi-Factor Authentication (MFA)**. | **The most effective measure.** Even if the password is stolen, the attacker will not have the second factor (token, biometric, etc.). |
| **Lack of Visibility** | Implement **Monitoring and Alerts** for multiple failed login attempts. | Allows the security team to detect and respond to an ongoing attack before it succeeds. |

--------------------------------------------------------------------------------------------------------------------------------------------------

## Conclusion

The lab highlighted critical vulnerabilities that compromise system security. The most urgent points for correction are:

### Critical Points to Be Corrected Immediately:

1.  **Default and Weak Credentials**
    - Remove default passwords like "msfadmin" and "password"
    - Implement a complex password policy with a minimum of 12 characters

2.  **Brute-Force Protection**
    - Configure automatic account lockout after 5 failed attempts
    - Implement progressive delays between login attempts

3.  **Outdated Services**
    - Update Apache 2.2.8 to a supported version
    - Update PHP 5.2.4 to the current version
    - Patch the vulnerable vsftpd 2.3.4

4.  **Web Security Configurations**
    - Implement security headers (CSP, X-Frame-Options)
    - Configure security flags on cookies (HttpOnly, Secure)
    - Remove sensitive files like phpinfo.php

5.  **Lack of Multi-Factor Authentication**
    - Implement MFA for administrative access
    - Add an extra layer of protection for critical logins

---

## License

This project is licensed under the MIT License. See the `LICENSE` file for more details.

---

## Author

**André Luís Alves Campos**

-------------------------------------------------------------------------

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/andlsac)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/andlsac)
[![Instagram](https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white)](https://instagram.com/andlsac)