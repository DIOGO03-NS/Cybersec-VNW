# Relatório de Auditoria de Segurança Ofensiva (Pentest)

-----

| **Parâmetro** | **Detalhes** |
| :--- | :--- |
| **Cliente** | TechCorp Solutions |
| **Alvo Principal** | Infraestrutura & Web Host (IP: 98.95.207.28) |
| **Modalidade** | Black Box (Simulação de Ameaça Externa) |
| **Data da Execução** | 30/11/2025 |
| **Consultor Líder** | Diogo Nogueira |
| **Classificação** | **Confidencial - Nível 1** |

-----

## 1\. Sumário Executivo

Este documento apresenta os resultados técnicos da avaliação de segurança realizada no ambiente da **TechCorp Solutions**. O objetivo da auditoria foi identificar, explorar e documentar vetores de ataque que poderiam comprometer a confidencialidade, integridade e disponibilidade dos ativos corporativos.

**Resumo da Avaliação:**
O nível de segurança do ambiente foi classificado como **CRÍTICO**.
A equipe de auditoria obteve êxito em comprometer a infraestrutura em sua totalidade. A partir de vulnerabilidades na camada Web, foi possível realizar movimentação lateral para o sistema operacional, culminando na exfiltração de dados sensíveis e obtenção de persistência administrativa.

**Estatísticas de Vulnerabilidades:**

  * 🔴 **Críticas:** 3 (Comprometimento total do sistema/dados)
  * 🟠 **Altas:** 2 (Acesso indevido/Modificação de dados)
  * 🟡 **Médias:** 2 (Exposição de informações)

-----

## 2\. Metodologia e Padrões

Os testes foram conduzidos seguindo as diretrizes do **PTES (Penetration Testing Execution Standard)** e focados nas vulnerabilidades listadas no **OWASP Top 10**.

A cadeia de ataque (*Cyber Kill Chain*) foi executada nas seguintes etapas:

1.  **Reconhecimento:** Coleta passiva e ativa de informações (OSINT).
2.  **Exploração:** Identificação e uso de vetores de entrada (CWE-89, CWE-79).
3.  **Pós-Exploração:** Enumeração local, escalação de privilégios e manutenção de acesso.

-----

## 3\. Detalhamento Técnico dos Achados

Abaixo estão descritas as vulnerabilidades exploradas, organizadas por ordem cronológica de execução do ataque.

### 3.1. Exposição de Dados Sensíveis (Information Disclosure)

**Severidade:** 🟡 **MÉDIA** | **CWE-200**

**Descrição:**
O servidor web apresenta falhas de configuração que permitem a indexação e leitura de arquivos que não deveriam ser públicos. Foram identificados artefatos de controle de versão (`.git`), backups de arquivos PHP e comentários de desenvolvimento em produção.

**Impacto de Negócio:**
Vazamento de lógica de negócio, estrutura de diretórios e, mais gravemente, credenciais hardcoded e tokens de API, facilitando ataques direcionados.

**Evidências:**

> *Comentários de desenvolvimento em produção:* <br>\<img src="Prints/Captura%20de%20tela%202025-11-30%20203419.png" width="500" style="border:1px solid \#ddd; border-radius:4px;" /\>

> *Mapeamento via Robots.txt:* <br>\<img src="Prints/Captura%20de%20tela%202025-11-30%20203519.png" width="500" style="border:1px solid \#ddd; border-radius:4px;" /\>

> *Vazamento de Tokens (Git/Config):* <br>\<img src="Prints/Captura%20de%20tela%202025-11-30%20203631.png" width="450" /\> \<img src="Prints/Captura%20de%20tela%202025-11-30%20203655.png" width="450" /\>

-----

### 3.2. Falhas de Injeção (SQLi & XSS)

**Severidade:** 🔴 **CRÍTICA** | **CWE-89, CWE-79**

**Descrição:**
A aplicação não realiza a sanitização adequada (Input Validation) nos campos de entrada.

1.  **SQL Injection:** O formulário de login permite a manipulação da query SQL, viabilizando o bypass de autenticação (`' OR '1'='1`).
2.  **Cross-Site Scripting (Reflected):** O campo de busca reflete scripts arbitrários, permitindo execução de código no lado do cliente.

**Impacto de Negócio:**
Acesso administrativo não autorizado sem necessidade de credenciais e risco de sequestro de sessão de usuários legítimos.

**Evidências:**

> *Payload de Bypass de Autenticação:* <br>\<img src="Prints/Captura%20de%20tela%202025-11-30%20203923.png" width="500" style="border:1px solid \#ddd; border-radius:4px;" /\>

> *Acesso Administrativo Concedido:* <br>\<img src="Prints/Captura%20de%20tela%202025-11-30%20203933.png" width="500" style="border:1px solid \#ddd; border-radius:4px;" /\>

> *Prova de Conceito (PoC) XSS:* <br>\<img src="Prints/Captura%20de%20tela%202025-11-30%20204004.png" width="450" /\> \<img src="Prints/Captura%20de%20tela%202025-11-30%20204013.png" width="450" /\>

-----

### 3.3. Configuração Insegura de Serviço (FTP)

**Severidade:** 🔴 **CRÍTICA** | **CWE-287**

**Descrição:**
O serviço FTP permite autenticação anônima. A auditoria no diretório raiz do serviço revelou arquivos contendo senhas em texto claro (`passwords.txt`), uma violação grave de políticas de segurança.

**Impacto de Negócio:**
Comprometimento total de credenciais que dão acesso a outros serviços críticos da infraestrutura (SSH, Banco de Dados).

**Evidências:**

> *Acesso Anônimo e Listagem:* <br>\<img src="Prints/Captura%20de%20tela%202025-11-30%20205827.png" width="500" style="border:1px solid \#ddd; border-radius:4px;" /\>

> *Exfiltração de Credenciais:* <br>\<img src="Prints/Captura%20de%20tela%202025-11-30%20212008.png" width="450" /\> \<img src="Prints/Captura%20de%20tela%202025-11-30%20212710.png" width="450" /\>

-----

### 3.4. Acesso Shell e Enumeração Local (SSH)

**Severidade:** 🟠 **ALTA** | **CWE-521**

**Descrição:**
Utilizando as credenciais obtidas no FTP, foi possível estabelecer conexão SSH na porta não-padrão **2222**. A análise interna (Post-Exploitation) revelou histórico de comandos não limpo (`.bash_history`) e arquivos de texto contendo segredos de negócio.

**Impacto de Negócio:**
Acesso direto ao sistema operacional com privilégios de usuário, servindo de base para escalação de privilégios e ataques à rede interna.

**Evidências:**

> *Conexão SSH Bem-sucedida:* <br>\<img src="Prints/Captura%20de%20tela%202025-11-30%20213610.png" width="500" style="border:1px solid \#ddd; border-radius:4px;" /\>

> *Enumeração de Arquivos Sensíveis:* <br>\<img src="Prints/Captura%20de%20tela%202025-11-30%20214011.png" width="450" /\> \<img src="Prints/Captura%20de%20tela%202025-11-30%20215220.png" width="450" /\>

-----

### 3.5. Escalação de Privilégios e Exfiltração de DB

**Severidade:** 🔴 **CRÍTICA** | **CWE-269**

**Descrição:**
O usuário comprometido possuía permissões de `sudo` excessivas. Através da análise de um script de automação (`backup_script.sh`), recuperaram-se credenciais de **root** do MySQL.
Isso permitiu o dump completo de tabelas ocultas (`secret_data`) e a injeção de novos administradores na aplicação web para garantir persistência.

**Evidências:**

> *Escalação via Script Vulnerável:* <br>\<img src="Prints/Captura%20de%20tela%202025-11-30%20224027.png" width="500" style="border:1px solid \#ddd; border-radius:4px;" /\>

> *Exfiltração da Base de Dados:* <br>\<img src="Prints/Captura%20de%20tela%202025-11-30%20222343.png" width="450" /\> \<img src="Prints/Captura%20de%20tela%202025-11-30%20222512.png" width="450" /\>

> *Persistência (Criação de Superadmin):* <br>\<img src="Prints/Captura%20de%20tela%202025-11-30%222908.png" width="500" style="border:1px solid \#ddd; border-radius:4px;" /\>

-----

## 4\. Plano de Ação e Recomendações

Dada a criticidade dos achados, recomenda-se a adoção imediata das seguintes medidas de correção, priorizadas por impacto:

### 4.1. Curto Prazo (Imediato)

1.  **Sanitização do Ambiente:** Remover arquivos `.git`, `.txt`, `.bak` e scripts de backup antigos dos diretórios públicos (`/var/www/html`).
2.  **Hardening de Serviços:**
      * Desabilitar acesso anônimo no FTP.
      * Desabilitar login via senha no SSH e impor uso de Chaves RSA/Ed25519.
3.  **Rotação de Segredos:** Alterar **todas** as senhas de banco de dados, usuários do sistema e chaves de API expostas.

### 4.2. Médio Prazo (Estrutural)

1.  **Correção de Código (Secure Coding):** Implementar *Prepared Statements* (PDO) em todas as consultas SQL e *Output Encoding* para prevenir XSS.
2.  **Gestão de Segredos:** Implementar cofre de senhas (ex: HashiCorp Vault) e remover credenciais hardcoded de scripts e códigos-fonte.
3.  **Princípio do Menor Privilégio:** Revisar regras do `sudoers` e permissões de arquivos críticos.

-----

**Documento gerado por Consultoria de Segurança Ofensiva.**
*Data: 30/11/2025*