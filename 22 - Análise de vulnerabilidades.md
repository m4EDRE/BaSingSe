
# Análise de vulnerabilidades

**`Objetivo da página:`**

- Identificar vulns de OS
- Identificar vulns de aplicações

**`Processo modelo (não existe receita de bolo):`**

- Identificar OS
- Identificar versão das aplicações e softwares
- Buscar por **vulns** conhecidas pra essas versões e **exploits**
- Busca por vulnerabilidades ainda não conhecidas
- Teste de ataques (bruteforce e etc)

**`Base de dados:`**

- [exploit-db.com](http://exploit-db.com)
- [securityfocus.com](http://securityfocus.com)
- [cvedetails.com](http://cvedetails.com)
- [nvd.nist.gov](http://nvd.nist.gov)
- [rapid7.com](http://rapid7.com) (metasploit)
- [packetstormsecurity.com](http://packetstormsecurity.com)

Off topic: `nmap -v —open -p <porta> <IP> —script=vuln -Pn`

Pode ajudar a encontrar vulnerabilidades

# Pesquisa manual por vulnerabilidades

O primeiro passo nesse tópico é aplicar tudo o que foi visto nos módulos anteriores de **Scanning, Information Gathering e Enumeration** para encontrar versões de todos os itens possíveis do alvo.

Feito isso partiremos para buscas nas bases de dados pelas versões encontradas nas fases anteriores

**`Exemplo:`** Foi identificado a aplicação webadmin em um alvo que é um site na internet varrendo suas portas.

Entraremos nas bases de dados [nvd.nist.gov](http://nvd.nist.gov) e [securityfocus.com](http://securityfocus.com) por exemplo e faremos uma busca por webadmin buscando vulns que se encaixem no cenário.

*Google Dorks podem ser úteis quando aplicadas para os sites de bases de dados

`searchsploit <item>` : Comando do Kali que auxilia na busca por vulns


---





# Scanners de Vulnerabilidades

Scanners de vulnerabilidades são ferramentas automatizadas que podem ser utilizadas para exploração de um grande número de vulnerabilidades ao mesmo tempo a partir de sua base de **plugins.**

`Principais Scanners`

- Nessus (Líder)
- Qualys
- OpenVas (Open Source)

Scanners podem ser utilizadas de diversas formas como identificação de hosts ativos, busca de portscans, identificação de OS e serviços entre outras funções

### Vantagens e Desvantagens

- Vantagens:
    
    - Teste de centenas de vulns em poucos segundos
    - Ajuda a identificar upgrade de patches
    - Ajuda para processos de auditoria
- Desvantagens
    
    - Não tem pós exploração
    - Gera falsos positivos
    - Gera falsos negativos
    - Gera ruído na rede
    
    *Por isso é sempre aliar isso a pesquisa manual


---




# Trabalhando com Nessus

**Download:**

- [tenable.com/downloads](http://tenable.com/downloads)
- baixou pro teu kali usa —> apt install ./`<arquivo>`

**Inicialização:**

- /etc/init.nessusd ou o systemctl start nessusd
    
- Porta para acesso via GUI: **8834**
    
    ### Scan básico
    
    A seguir estão os itens a serem selecionados ao fazer um scan básico, alguns itens estão disponíveis em outros scans e assim essa explicação também conta para eles.
    
    Selecione new Scan —> Basic Scan:
    
    - **Nome do Scan**
    - **Targets**: Alvo a ser revisado pelo Scan
    - **Schedule e Notificações**: Quando fazer e quem notificar via e-mail
    - **Tipo de Portscan**: SYN, All ports, Common Ports….
    - **Forma de descoberta de hosts:** (Pn)
    - **Tipo de Scan**: Aprofundado, Web, Simples…
    - **Forma de Report**
    
    *Caso tenha: **Credenciais para autenticação no host e plugins a serem usados**
    
    ### Análise de aplicações WEB
    
    Template: **Web Application Test**
    
    - Nome do Scan
    - Target
    - - Aba **Assessements** deve ser preenchida
    - - Plugins escolhidos individualmente na aba Plugins
    
    ### Patch Assessment
    
    Auditoria por patches a serem atualizados no sistema….
    
    Nessus faz login na máquina a revisa patches que podem ter upgrade conforme sua base de dados.
    
    Template: **Credentialed patched audit**
    
    - **Nome do Scan**
    - **Target**
    - **Credenciais**
    - **Não alterar as outras opções!**
    
    ### Ataques de força Bruta via Nessus
    
    Template: Scan avançado
    
    *Necessita do arquivo de users e passwords
    
    *Tirar o máximo de opções possíveis pra evitar detecção
    
    - **Nome do Scan**
    - **Target**
    - **Aba Discovery**: (Tira o ping)
    - **Portas**: Tira o máximo de opções que puder
    - **Aba Assessment**:
        - Bruteforce
        - Habilita Hydra
        - Seleciona opções pro bruteforce
    - **Plugins**: Apenas plugins de força bruta
    
    ### Scan Avançado (Like a Pro)
    
    1. Acessar [tenable.com/plugins](http://tenable.com/plugins) e fazer a análise dos plugins para a aplicação que você quer analisar, olhar a familia do plugin
    2. Em caso de IPS filtrando o Scan você faz primeiro o Bypass manual, acha as portas e coloca as portas manualmente no scan automatizado
    3. Montar todo o Scan conforme os outros exemplos acima, lembrando de especificar as familias corretas para explorar as aplicações



---





# Compliance PCI-DSS

- **`PCI-DSS`**: Criado por empresas de cartão de crédito, é uma certificação que tem requisitos especificos de segurança para manter um padrão estabelecido a empresas que geram informações muito sensíveis
    
    Auditor que chega para essa validação pede diversos documentos, inclusive o de pentest interno e externo conforme as normas
    
    **Guia para pentesters no site PCI-DSS: [https://listings.pcisecuritystandards.org/documents/Penetration-Testing-Guidance-v1_1.pdf](https://listings.pcisecuritystandards.org/documents/Penetration-Testing-Guidance-v1_1.pdf)
    
    ### Scan para PCI-DSS
    
    Template: PCI-DSS
    
    - Família do item Scan deve ser: Compliance
        
        Esse scan retorna se você passaria na verificação ou não!
        


---





# Falsos Positivos e Dicas

O grande problema dos scans automáticos sempre é a quantidade de falsos positivos que inevitavelmente serão gerados analisando os diversos alvos do ambiente, para isso, a solução é sempre aliar o automático ao manual em termos de análise de vulnerabilidades

### Dicas:

- Sempre equilibrar automático com manual
- Realizar POCs após os Scans sempre para confirmar que o alvo é explorável daquela forma
- Sempre revise todas as informações passadas ao cliente. Isto é, jamais passe informações que o Scan automatizado coletou e você não leu e confirmou o que é

---





# NMAP NSE (Identificação de vulns)

Abaixo revisaremos a opção do NMAP que auxilia na identificação de vulnerabilidades, para isso utilizaremos scripts localizados em `/usr/share/nmap/scripts`

`nmap —script=<script> <alvo>` : Utiliza um script próprio do NMAP para identificar vulnerabilidades de uma aplicação ou de um OS.

`nmap —script=<script> --script-args <args> <alvo>` : Utiliza um script próprio do NMAP que necessita de argumentos para identificar vulnerabilidades de uma aplicação ou de um OS.

**Sempre revisar se o script precisa de argumentos para funcionar (nano do script)



----





# Shadow Brokers

O que é o Shadow Brokers?

Shadow Brokers foi um grupo hacker que ficou mundialmente famoso por vazar informações sobre hacking de alto nível de sofisticação pertencentes ao governo americano.

Entre essas informações estão os ultra conhecidos **`ETERNALS….`**

- **`EternalBlue`**
    - Explora o SMBv1
    - **Afeta:** Windows XP até Windows Server 2012 (antes do patch MS17-010).
    - **Uso notório:** base para o ransomware **WannaCry** e o malware **NotPetya**.
    - **Impacto:** paralisou hospitais, empresas e órgãos públicos em todo o mundo.
- **`EternalChampion`**
    - **Explora:** também SMBv1, mas usando técnicas diferentes.
    - **Função:** execução remota de código (RCE).
    - **Combina com:** outras ferramentas como **DoublePulsar** para implantar malwares.
- **`EternalRomance`**
    - **Explora:** SMB em sistemas Windows, com foco em versões antigas.
    - **Uso notório:** também foi usado em variantes do NotPetya.
    - **Permite:** execução de código remoto e escalonamento de privilégios.
- **`EternalSynergy`**
    - **Explora:** SMBv1 e SMBv2.
    - **Afeta:** versões de Windows mais recentes que EternalRomance.
    - **Técnica:** canal de comunicação malicioso que permite controlar o host.

Além dos "Eternals", o vazamento também incluiu:

- **DoublePulsar**: backdoor que pode ser implantado após os exploits.
- **Fuzzbunch**: uma espécie de Metasploit interno da NSA para orquestrar ataques com os Eternals.
- **Danderspritz**: suíte de pós-exploração para espionagem e controle do alvo.