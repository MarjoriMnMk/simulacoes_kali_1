# simulacoes_kali_1**
Simulações usando Medusa e Nmap**

**Simulações utilizando um alvo real, dentro do meu servidor**

# Identificando quais serviços estão instalados no alvo

```bash
[kali@customer ~]$ nmap -sV OCULTADO
```

Resultado:

```text
Starting Nmap 7.93 ( https://nmap.org ) at 2026-05-10 18:50 -03

Nmap scan report for OCULTADO (IP OCULTADO)
Host is up (0.20s latency).

Not shown: 962 filtered tcp ports (no-response),
34 closed tcp ports (conn-refused)

PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      ProFTPD or KnFTPD
80/tcp   open  http     LiteSpeed httpd
443/tcp  open  ssl/http LiteSpeed httpd
3306/tcp open  mysql?

Service Info: OS: Unix
```

## Complementando comando acima

```bash
nmap --script ssl-cert,ssl-enum-ciphers -p 443 OCULTADO

nmap --script ftp-anon -p 21 OCULTADO

nmap --script mysql-info -p 3306 OCULTADO
```

Resultado, 3 serviços ativos, o que é comum em ambientes de hospedagem web.
Encontrado Let's Encrypt, senhas receberam nota A, ambiente do banco de dados está exposto publicamente com a versão 11.8.6-MariaDB-log

Testes elaborados com o Medusa e site externo
```
medusa -h OCULTADO -U users.txt -P pass.txt -M http \ -m PAGE:'https://OCULTADO/#/login' \ -m FORM:'email=^USER^&password=^PASS' \ -m 'FAIL=Invalid email or password.' -t 6
```
Usuários encontrados: user, admin, root, teste com as senhas

**Testes de sql injection**
```
sqlmap -u 'OCULTADO/search?q=teste'
[*] starting @ 20:06:13 /2026-05-10/

[20:06:13] [INFO] testing connection to the target URL
[20:06:14] [INFO] testing if the target URL content is stable
[20:06:15] [INFO] target URL content is stable
[20:06:15] [INFO] testing if GET parameter 'utm_source' is dynamic
[20:06:16] [WARNING] GET parameter 'utm_source' does not appear to be dynamic
[20:06:17] [WARNING] heuristic (basic) test shows that GET parameter 'utm_source' might not be injectable
[20:06:18] [INFO] testing for SQL injection on GET parameter 'utm_source'
[20:06:18] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[20:06:22] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[20:06:23] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[20:06:27] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[20:06:30] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[20:06:34] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[20:06:38] [INFO] testing 'Generic inline queries'
[20:06:39] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[20:06:41] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[20:06:44] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[20:06:47] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[20:06:51] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[20:06:54] [INFO] testing 'Microsoft SQL Server/Sybase time-based blind (IF)'
[20:06:58] [INFO] testing 'Oracle AND time-based blind'
it is recommended to perform only basic UNION tests if there is not at least one other (potential) technique found. Do you want to reduce the number of requests? [Y/n] y
[20:07:15] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[20:07:23] [WARNING] GET parameter 'utm_source' does not seem to be injectable
[20:07:23] [CRITICAL] all tested parameters do not appear to be injectable. Try to increase values for '--level'/'--risk' options if you wish to perform more tests. If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') and/or switch '--random-agent'
```

```
sqlmap -u "https://OCULTADO/admin/index.php" --method POST --data='{"email":"?","senha":"?"}' --headers="Content-Type: application/json"
```

403 (Forbidden) - 120 times, 429 (Too Many Requests) - 24 times

Aqui o servidor interrompeu as requisições
Interessante ver o log do lado do servidor, apontando que houve a requisição pelo sqlmap
<img width="952" height="339" alt="image" src="https://github.com/user-attachments/assets/6c712ed2-b829-48a8-bbe3-7cde91668678" />

**Testes usando o wpscan**
```
wpscan --url OCULTADO
```
- WordPress 6.9.4 identificado (versão atualizada)

- `wp-cron.php` acessível externamente, podendo permitir consumo excessivo de recursos

- `xmlrpc.php` acessível

- `readme.html` acessível, podendo indicar informações de instalação

- `robots.txt` acessível, podendo indicar áreas ocultas

- Tema identificado: Royal Elementor Kit

- Alguns plugins levemente desatualizados

- Nenhum backup exposto encontrado
