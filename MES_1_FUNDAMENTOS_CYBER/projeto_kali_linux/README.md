# Projeto Mês 1 — Kali Linux: Ambiente e 10 Comandos Essenciais de Segurança

**Curso:** Linux para Cibersegurança: administração, shell scripting e Kali Linux — Alura  
**Mês:** 1 · Fundamentos de Cybersecurity  
**Data de conclusão:** Abril 2026

---

## Objetivo

Configurar um ambiente Kali Linux em máquina virtual e explorar as 10 ferramentas/comandos mais utilizados em segurança ofensiva, documentando uso prático, exemplos reais e conexão com o perfil de AI Security Engineer.

---

## Ambiente

| Item | Detalhe |
|------|---------|
| Distribuição | Kali Linux (Debian-based, foco em segurança ofensiva) |
| Virtualização | VirtualBox / VMware |
| Credenciais padrão | `kali` / `kali` (alterar em produção) |
| Por que VM? | Isolamento total: snapshots, pausar o sistema, sem risco ao host |

> **Boas práticas:** Nunca instalar Kali diretamente no computador pessoal. A VM permite reverter para um estado seguro caso algo dê errado durante testes.

---

## Os 10 Comandos/Ferramentas Essenciais

### 1. `nmap` — Escaneamento de Portas e Serviços

**O que faz:** Descobre hosts ativos, portas abertas e versões de serviços em uma rede.

```bash
# SYN stealth scan nas 100 portas mais comuns (menos ruidoso)
nmap -v -sS --top-ports=100 192.168.1.0/24

# Detecção de versão de serviços em portas específicas
nmap -v -sV -p22,80,443 192.168.1.10
```

**Casos de uso em AI Security:** Mapear a superfície de ataque de endpoints de inferência de modelos ML expostos via API.

---

### 2. `whois` — Reconhecimento de Domínios

**O que faz:** Enumera informações de registro de domínio: proprietário, CNPJ, nameservers, datas de criação/expiração.

```bash
whois alura.com.br
```

**Output relevante:** Nameservers, contatos administrativos, faixas de IP registradas. Fase de *information gathering* no ciclo de pentest.

---

### 3. `dnsenum` — Enumeração de DNS

**O que faz:** Coleta informações de DNS — servidores de e-mail (MX), nameservers, subdomínios e ranges de IP.

```bash
dnsenum alura.com.br
```

**Por que importa:** Subdomínios expostos podem revelar ambientes de homologação ou APIs internas sem proteção adequada.

---

### 4. `theHarvester` — OSINT de E-mails e Subdomínios

**O que faz:** Coleta e-mails de funcionários, subdomínios, IPs e hosts a partir de fontes públicas (OSINT).

```bash
theharvester -d alura.com.br -l 500 -b hackertarget
```

**Conexão AI Security:** Funcionários com e-mails expostos são vetores de phishing — exatamente o problema que o projeto do Mês 5 vai resolver com ML.

---

### 5. `nikto` — Scanner de Vulnerabilidades Web

**O que faz:** Varredura detalhada em aplicações web identificando vulnerabilidades sem executar ataques.

```bash
nikto -url http://192.168.1.10
```

**O que o Nikto encontra:**
- Servidor desatualizado (ex: Apache 2.4.41 sem suporte)
- Cookies sem flag `HttpOnly` (vetor XSS)
- Cabeçalhos de segurança ausentes
- Diretórios indexados (`/config`, `/style`)
- Arquivos sensíveis expostos (`.git`, XMLs)

---

### 6. `searchsploit` — Busca de Exploits Offline

**O que faz:** Pesquisa no banco de dados local do Exploit-DB por exploits conhecidos para serviços e versões específicas.

```bash
searchsploit apache 2.4
searchsploit -x exploits/linux/remote/12345.py
```

**Fluxo típico:** `nmap` identifica a versão → `searchsploit` busca CVEs e exploits disponíveis → análise de impacto.

---

### 7. `john` (John the Ripper) — Quebra de Hashes

**O que faz:** Tenta descobrir senhas originais a partir de hashes usando wordlists ou força bruta.

```bash
# Quebrar hash MD5
john --format=Raw-MD5 hash.txt

# Exibir senha descoberta
john --show hash.txt
```

**Por que estudar isso:** Entender como hashes são quebrados ensina a escolher algoritmos de armazenamento seguros (bcrypt, Argon2) — fundamental para proteger credenciais em sistemas de IA.

---

### 8. `hydra` — Brute Force em Formulários e Protocolos

**O que faz:** Automatiza tentativas de login em formulários web, SSH, FTP e outros protocolos usando wordlists.

```bash
# Brute force em formulário HTTP POST
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  192.168.1.10 http-post-form \
  "/login.php:user=^USER^&pass=^PASS^:Invalid credentials"

# Brute force SSH
hydra -l root -P wordlist.txt ssh://192.168.1.10
```

**Fluxo de uso:** Capturar requisição HTTP POST no DevTools do navegador → extrair parâmetros → montar comando Hydra.

---

### 9. `grep` + `awk` — Análise de Logs

**O que faz:** Ferramentas de linha de comando para filtrar, extrair e processar logs de segurança.

```bash
# Filtrar tentativas de login falhas
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr

# Extrair IPs únicos de um log de acesso web
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20
```

**Conexão AI Security:** Análise de logs é a base do detector de anomalias que será construído no Mês 5.

---

### 10. `chmod` / `chown` — Controle de Permissões

**O que faz:** Define quem pode ler, escrever e executar arquivos e diretórios no Linux.

```bash
# Remover permissão de escrita para outros usuários
chmod o-w arquivo_sensivel.conf

# Permissão apenas para o dono (ideal para chaves SSH)
chmod 600 ~/.ssh/id_rsa

# Transferir propriedade de arquivo
chown usuario:grupo arquivo.py
```

**Conexão AI Security:** Datasets de treino e modelos serializados (`.pkl`, `.pt`) devem ter permissões restritivas — um arquivo de modelo com permissão `777` é uma superfície de ataque.

---

## Conceitos Fundamentais Estudados

### Tríade CIA
| Pilar | Definição | Exemplo Prático |
|-------|-----------|-----------------|
| **Confidencialidade** | Acesso restrito a pessoas autorizadas | Autenticação + MFA em APIs de inferência |
| **Integridade** | Dados completos e sem alterações não autorizadas | SHA-256 para verificar datasets de treino |
| **Disponibilidade** | Acesso garantido quando necessário | Redundância em serviços de modelo em produção |

### Sistema de Classificação de Vulnerabilidades
- **CVE** — ID único por vulnerabilidade (ex: `CVE-2021-44228` = Log4Shell)
- **CWE** — Categorias de fraquezas de software (ex: CWE-79 = XSS)
- **CVSS 4.0** — Pontuação contextual de severidade (0.0 a 10.0)

### Fases do Hacking Ético
1. **Reconhecimento** — whois, dnsenum, theHarvester
2. **Escaneamento** — nmap, nikto
3. **Análise de vulnerabilidades** — searchsploit, nikto
4. **Exploração** (autorizada) — hydra, john, metasploit
5. **Relatório** — documentação de impacto e remediação

---

## Conexão com IA (FIAP)

| Conceito de Security | Aplicação em IA |
|---------------------|-----------------|
| Confidencialidade de dados | Acesso restrito a datasets de treino |
| Integridade (SHA-256) | Verificação de checksum de modelos baixados |
| Análise de logs com grep/awk | Base de detectores de anomalia com ML |
| Permissões Linux | Proteção de arquivos `.pkl` e `.env` com chaves de API |
| Information gathering | Superfície de ataque de endpoints de inferência |

---

## Certificado

[Curso Linux para Cibersegurança — Alura](./../../MES_1_FUNDAMENTOS_CYBER/cursos/Linux%20para%20cibersegurança%20administração%2C%20shell%20scripting%20e%20Kali%20Linux/Victor%20Gonçalves%20-%20Curso%20Linux%20para%20cibersegurança_%20administração%2C%20shell%20scripting%20e%20Kali%20Linux%20-%20Alura.pdf)

---

> **Aviso:** Todas as ferramentas documentadas aqui foram estudadas em ambiente isolado (VM) para fins educacionais. O uso dessas ferramentas em sistemas sem autorização explícita é ilegal.
