# PROPOSTA DE CONSULTORIA DE SEGURANÇA CIBERNÉTICA - OPÇÃO 2
> **Cliente:** LojaZeta
> **Data:** 25/09/2025

## 1. Sumário Executivo

A LojaZeta, um e-commerce em franco crescimento, enfrenta riscos cibernéticos significativos que ameaçam sua operação, reputação e a confiança de seus clientes. Incidentes recentes, como tentativas de SQL Injection, Cross-Site Scripting (XSS) e ataques de força bruta, são indicadores claros de que a atual infraestrutura, sem defesas dedicadas e monitoramento centralizado, está vulnerável a ataques comuns e destrutivos. A ausência de um SIEM e de testes de restauração de backup agrava o cenário, tornando a detecção e a recuperação de um incidente lento e incerto.

Esta proposta apresenta uma estratégia de segurança em profundidade, pragmática e de baixo custo, projetada para a realidade da LojaZeta. O plano se baseia em três pilares:

1.  **Arquitetura de Defesa em Camadas:** Implementação de controles essenciais desde o perímetro até os dados, com foco em um Web Application Firewall (WAF) para proteger contra ataques web e um sistema de Identity and Access Management (IAM) robusto para mitigar acessos não autorizados.
2.  **Monitoramento Centralizado e Acionável:** Adoção de uma solução SIEM open-source (Wazuh) para agregar logs de todas as fontes críticas (Nginx, Node.js, PostgreSQL, SO), permitindo a criação de alertas automáticos para atividades suspeitas e fornecendo visibilidade unificada.
3.  **Resposta a Incidentes Simplificada:** Estruturação de um plano de resposta a incidentes baseado no framework NIST, com procedimentos claros e um roadmap de implementação priorizado (80/20) para garantir *quick wins* de segurança em 30 dias.

A implementação desta proposta elevará drasticamente a maturidade de segurança da LojaZeta, reduzindo a superfície de ataque, melhorando a capacidade de detecção e garantindo uma resposta rápida e eficaz a futuros incidentes, tudo isso com um investimento consciente e alinhado ao tamanho da equipe.

---

## 2. Escopo e Metodologia

O escopo deste trabalho cobre a infraestrutura em nuvem (IaaS) da LojaZeta, incluindo seu stack tecnológico (Nginx, Node.js, PostgreSQL) e os processos operacionais de segurança. A metodologia adotada foi uma análise de risco baseada no briefing fornecido, priorizando os requisitos funcionais e não funcionais do cliente e aplicando o princípio de Pareto (80/20) para focar nas ações de maior impacto com menor esforço inicial.

**Suposições:**
* A infraestrutura está hospedada em um provedor de nuvem moderno (ex: AWS, GCP, Azure) que oferece serviços básicos de segurança (ex: Security Groups, WAF).
* A equipe da LojaZeta (1 ops) terá disponibilidade para implementar as recomendações com o suporte desta consultoria.
* O orçamento para novas ferramentas é limitado, priorizando soluções open-source ou serviços gerenciados de baixo custo.

---

## 3. Arquitetura de Defesa (Camadas)

A defesa em profundidade garante que a falha de um único controle não resulte em um comprometimento total. Propomos a seguinte arquitetura:

* **Diagrama Visual:** Veja o arquivo `diagrama.png` para uma representação gráfica detalhada.

* **Camada 1: Perímetro**
    * **Controle:** Web Application Firewall (WAF) e CDN.
    * **Proposta:** Ativar o WAF do provedor de nuvem (ex: AWS WAF, Cloudflare) com o conjunto de regras gerenciadas da OWASP (OWASP Top 10) em modo **Blocking**. Isso irá mitigar imediatamente as tentativas de SQLi e XSS. A CDN (ex: Cloudflare, AWS CloudFront) também ajudará a proteger contra ataques de negação de serviço (DDoS).

* **Camada 2: Rede**
    * **Controle:** Segmentação de Rede e Controle de Acesso.
    * **Proposta:** Utilizar os recursos de rede da nuvem (ex: Security Groups, NACLs) para segmentar o ambiente.
        * A instância do Nginx deve ser a única acessível publicamente na porta 443 (HTTPS).
        * A instância da aplicação (Node.js) só deve aceitar tráfego vindo da instância do Nginx.
        * O banco de dados (PostgreSQL) só deve aceitar tráfego da instância da aplicação.
        * Acesso administrativo (SSH) deve ser restrito a IPs conhecidos (VPN do escritório).

* **Camada 3: Host (Sistema Operacional)**
    * **Controle:** Hardening e Monitoramento de Host.
    * **Proposta:**
        * Implementar agentes do SIEM (Wazuh) em todas as instâncias para monitorar a integridade de arquivos, detectar processos maliciosos e coletar logs.
        * Remover software desnecessário das instâncias.
        * Manter o sistema operacional e os pacotes sempre atualizados.

* **Camada 4: Aplicação**
    * **Controle:** Hardening da Aplicação e Gestão de Dependências.
    * **Proposta:**
        * Implementar *rate limiting* no Nginx e na rota `/login` para dificultar ataques de força bruta.
        * Realizar scans de vulnerabilidade nas dependências do Node.js (ex: `npm audit`).
        * Garantir que os logs da aplicação sejam estruturados (JSON) para facilitar a análise no SIEM.

* **Camada 5: Dados**
    * **Controle:** Backup e Criptografia.
    * **Proposta:**
        * Automatizar e **testar regularmente** o processo de restauração dos backups do PostgreSQL.
        * Criptografar os dados em repouso (volumes do banco de dados) e em trânsito (TLS/SSL).

* **Camada 6: Identidade**
    * **Controle:** Autenticação Forte e Mínimo Privilégio.
    * **Proposta:**
        * Habilitar **Autenticação de Múltiplos Fatores (MFA)** para o acesso ao painel do provedor de nuvem e para o acesso SSH.
        * Criar políticas de senhas fortes para a aplicação.
        * Implementar bloqueio de conta após múltiplas tentativas de login falhas.

---

## 4. Monitoramento & SIEM

A falta de visibilidade é um dos maiores riscos. Propomos a implementação de uma solução SIEM centralizada e de baixo custo.

* **Ferramenta Sugerida:** **Wazuh**. É uma plataforma open-source de detecção de intrusão e monitoramento de segurança que já vem com regras para análise de logs, verificação de integridade e resposta a ameaças.

* **Fontes de Log a Coletar:**
    * **Nginx:** Logs de acesso (`access.log`) e erro (`error.log`).
    * **Node.js:** Logs da aplicação (requisições, erros, eventos de autenticação).
    * **PostgreSQL:** Logs de queries, erros e conexões.
    * **Sistema Operacional:** Logs de autenticação (`auth.log`), syslog e logs de auditoria do sistema.

* **Casos de Uso e Alertas (Exemplos):**
    1.  **Detecção de SQL Injection:** Criar regra no Wazuh para alertar quando padrões de SQLi (ex: `UNION SELECT`, `' OR 1=1`) forem detectados nos logs de acesso do Nginx ou da aplicação.
    2.  **Detecção de XSS:** Alertar sobre a presença de tags `<script>` e outros vetores de XSS em parâmetros de URL ou corpos de requisição.
    3.  **Múltiplas Falhas de Login:** Alertar quando um mesmo IP gerar mais de 10 tentativas de login com falha em menos de 5 minutos na rota `/login`.
    4.  **Acesso SSH de IP Desconhecido:** Gerar alerta crítico se uma conexão SSH for estabelecida a partir de um IP não autorizado.

* **KPIs (Key Performance Indicators):**
    * **MTTD (Mean Time to Detect):** Tempo médio para detectar um incidente (meta: < 1 hora).
    * **MTTR (Mean Time to Respond):** Tempo médio para responder a um alerta (meta: < 30 minutos para alertas críticos).
    * **Tentativas de Ataque Bloqueadas:** Número de ataques bloqueados pelo WAF por dia.

---

## 5. Resposta a Incidentes (NIST IR)

Um plano simples e claro é fundamental para uma equipe pequena.

* **Fase 1: Preparação**
    * **Ações:** Instalar e configurar o Wazuh, definir papéis (quem é o líder do incidente, quem executa as ações técnicas), garantir acesso rápido aos backups.

* **Fase 2: Detecção e Análise**
    * **Ações:** O analista de plantão recebe o alerta do Wazuh via e-mail ou Slack. Ele acessa o dashboard do Wazuh para validar o alerta, verificar o IP de origem, o alvo e o tipo de ataque. Usando a Matriz de Classificação, ele define a severidade (ex: tentativa de SQLi = Alta).

* **Fase 3: Contenção, Erradicação e Recuperação**
    * **Contenção:**
        * Para ataques web: Bloquear o IP do atacante no WAF ou no Security Group.
        * Para um host comprometido: Isolar a instância da rede alterando seu Security Group.
    * **Erradicação:**
        * Identificar a causa raiz (ex: vulnerabilidade no código, senha fraca).
        * Aplicar a correção (ex: deploy de código corrigido, reset de senha).
    * **Recuperação:**
        * Restaurar a aplicação ou o banco de dados a partir do último backup testado e validado.
        * Monitorar intensivamente os logs para garantir que a ameaça foi eliminada.

* **Fase 4: Lições Aprendidas**
    * **Ações:** Realizar uma reunião pós-incidente para discutir o que funcionou, o que falhou e como melhorar. Atualizar as regras do SIEM e os procedimentos de segurança.

* **Runbooks:** Criar documentos simples em `runbooks/` para os incidentes mais comuns (SQLi, brute-force), detalhando o passo a passo da contenção.

---

## 6. Recomendações (80/20) e Roadmap

| Fase | Duração | Ações (80/20 - Foco em Quick Wins) | Responsável |
| :--- | :--- | :--- | :--- |
| **Fase 1: Quick Wins** | **30 Dias** | 1. Ativar WAF do provedor de nuvem em modo **Blocking**.<br>2. Revisar e aplicar regras restritivas de Security Groups.<br>3. Implementar **MFA** no acesso ao provedor de nuvem e SSH.<br>4. Configurar *rate limiting* para a rota `/login`. | Time de Ops |
| **Fase 2: Estruturação** | **3-6 Meses** | 1. Implementar o SIEM **Wazuh** e coletar logs das fontes críticas.<br>2. Criar os 3 alertas principais (SQLi, XSS, Brute-force).<br>3. Automatizar e **testar o procedimento de restauração de backup**.<br>4. Iniciar scans de vulnerabilidade automatizados (ex: Trivy, npm audit). | Time de Ops & Devs |
| **Fase 3: Maturidade** | **6-12 Meses**| 1. Desenvolver runbooks detalhados para os principais tipos de incidentes.<br>2. Realizar um treinamento básico de conscientização em segurança para os desenvolvedores.<br>3. Contratar um teste de invasão (pentest) básico para validar as defesas. | Gestão & Time Completo |

---

## 7. Riscos, Custos e Suposições

* **Riscos:**
    * A equipe enxuta pode se tornar um gargalo na implementação.
    * Falsos positivos no WAF ou SIEM podem gerar sobrecarga operacional.
    * A falta de um orçamento dedicado pode impedir a adoção de ferramentas mais avançadas no futuro.
* **Custos:**
    * **WAF:** Custo variável dependendo do tráfego (geralmente baixo para o porte da LojaZeta em provedores de nuvem).
    * **SIEM (Wazuh):** Custo de uma instância para hospedar o servidor Wazuh. O software é gratuito.
    * **Hora/Homem:** Principal custo será o tempo da equipe para implementação e monitoramento.
* **Suposições:**
    * A equipe possui conhecimento básico em administração de sistemas Linux e redes em nuvem.
    * Há comprometimento da gestão para investir o tempo necessário nas melhorias de segurança.

---

## 8. Conclusão

A LojaZeta está em um ponto de inflexão onde o investimento em segurança não é mais opcional, mas uma necessidade para a sustentabilidade do negócio. A estratégia delineada nesta proposta oferece um caminho claro, pragmático e de alto impacto para fortalecer as defesas, obter visibilidade e preparar a empresa para responder a incidentes de forma eficaz, transformando a segurança de uma fraqueza em uma vantagem competitiva.

**Próximos Passos:**
1.  Aprovação da proposta.
2.  Reunião de kickoff para detalhar o plano de 30 dias.
3.  Início da implementação da Fase 1 (Quick Wins).