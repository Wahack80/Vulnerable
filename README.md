# 🔐 HackLab Academy — Entorno de Práctica Pentesting Web

> **⚠ AVISO LEGAL:** Esta plataforma contiene vulnerabilidades **intencionales** para fines exclusivamente educativos.  
> Aplicar estas técnicas en sistemas sin autorización es un **delito penal**. Úsala únicamente en entornos propios o con permiso escrito.

---

## ¿Qué es HackLab Academy?

HackLab Academy es una plataforma web de práctica para estudiantes de **ciberseguridad y ethical hacking**. Contiene **10 módulos de vulnerabilidades** basados en el **OWASP Top 10 2021**, con teoría, laboratorios interactivos, pistas y flags de captura.

No requiere backend, base de datos ni instalación de dependencias. Es un único archivo HTML que funciona en cualquier servidor web estático.

---

## 📦 Contenido

```
hacklab-public/
├── index.html          ← Plataforma principal (todo en uno)
├── README.md           ← Este archivo
└── GUIA-PRACTICA.md    ← Guía paso a paso para estudiantes
```

---

## 🚀 Cómo Correrlo

### Opción 1 — Servidor local en Kali Linux (recomendado para práctica real)

```bash
# Clona o descarga el repositorio
cd hacklab-public/

# Sirve con Python (viene preinstalado en Kali)
python3 -m http.server 8080

# Abre en el navegador
http://localhost:8080
```

Ahora puedes atacarlo con **Burp Suite, sqlmap, nikto, curl** y cualquier herramienta de Kali.

### Opción 2 — Abrir directamente en el navegador

Simplemente abre `index.html` en tu navegador. Funciona sin servidor para explorar los módulos interactivos.

### Opción 3 — Hosting estático gratuito (público)

Puedes desplegarlo en plataformas gratuitas:

**GitHub Pages:**
```bash
git init
git add .
git commit -m "HackLab Academy"
git remote add origin https://github.com/TU_USUARIO/hacklab-academy.git
git push -u origin main
# Activa GitHub Pages en Settings → Pages → Branch: main
```

**Netlify (drag & drop):**
1. Ve a [netlify.com](https://netlify.com)
2. Arrastra la carpeta `hacklab-public/` al dashboard
3. Listo — URL pública en segundos

**Vercel:**
```bash
npm i -g vercel
vercel deploy
```

---

## 📚 Módulos Disponibles

| # | Módulo | OWASP | Dificultad | Técnicas |
|---|--------|-------|-----------|----------|
| 1 | SQL Injection | A03:2021 | 🟢 Fácil | Login bypass, UNION, Blind |
| 2 | XSS | A03:2021 | 🟢 Fácil | Reflejado, Almacenado, DOM |
| 3 | Autenticación Rota | A07:2021 | 🟢 Fácil | Credenciales débiles, Brute Force, Session Hijack |
| 4 | IDOR | A01:2021 | 🟡 Medio | Escalada horizontal, Path Traversal |
| 5 | JWT Attacks | A02:2021 | 🟡 Medio | alg:none, Weak secret, Token forgery |
| 6 | Command Injection | A03:2021 | 🟡 Medio | OS injection, Filter bypass |
| 7 | SSRF | A10:2021 | 🔴 Difícil | Metadata AWS, Servicios internos |
| 8 | XXE | A05:2021 | 🔴 Difícil | File read, XXE→SSRF |
| 9 | CSRF | A01:2021 | 🟡 Medio | Token bypass, Email takeover |
| 10 | LFI / Path Traversal | A01:2021 | 🟡 Medio | Dir traversal, Log poisoning → RCE |

---

## 🛠 Herramientas Recomendadas

### Interceptación y análisis
- **Burp Suite Community** — Interceptar, modificar y repetir requests HTTP
- **OWASP ZAP** — Alternativa gratuita a Burp Suite

### Inyección y fuzzing
- **sqlmap** — Automatizar SQL Injection: `sqlmap -u "http://localhost:8080/?id=1" --dbs`
- **ffuf / gobuster** — Fuzzing de directorios y parámetros
- **nikto** — Escáner de vulnerabilidades web: `nikto -h http://localhost:8080`

### Fuerza bruta
- **hydra** — Ataques de diccionario: `hydra -l admin -P rockyou.txt target http-post-form`
- **hashcat** — Crackeo de hashes y JWT secrets: `hashcat -a 0 -m 16500 token.jwt wordlist.txt`

### Análisis de tokens
- **jwt.io** — Decodificar y modificar JWTs manualmente
- **jwt_tool** — Automatizar ataques JWT: `python3 jwt_tool.py TOKEN -X a` (alg:none)

### Utilidades
- **curl** — Hacer requests personalizados desde terminal
- **Python requests** — Scripting de ataques
- **Burp Suite Repeater/Intruder** — Modificar y automatizar requests

---

## 🎯 Rutas de Aprendizaje

### Ruta Principiante (0-3 meses de experiencia)
```
SQL Injection → XSS Reflejado → Auth Bypass (credenciales) → IDOR
```

### Ruta Intermedia (3-12 meses)
```
JWT Attacks → Command Injection → CSRF → LFI/Path Traversal
```

### Ruta Avanzada (1+ años)
```
SSRF → XXE → Chaining (XSS+CSRF, SSRF+XXE, LFI→RCE)
```

### Para certificaciones
| Certificación | Módulos más relevantes |
|--------------|----------------------|
| eJPT (eLearnSecurity) | SQLi, XSS, Auth, IDOR, CMDi |
| CEH (EC-Council) | Todos los módulos |
| OSCP (OffSec) | CMDi, LFI→RCE, SSRF |
| CompTIA PenTest+ | SQLi, XSS, CSRF, JWT |
| BSCP (Burp Suite Certified) | Todos con énfasis en XXE, SSRF |

---

## 📋 Flags de Captura

Cada módulo tiene una o más flags en formato `FLAG{NOMBRE}`. Al explotarlos correctamente:

```
FLAG{SQL_LOGIN_BYPASS}
FLAG{SQL_UNION_DATA_EXFIL}
FLAG{SQL_BLIND_BOOLEAN}
FLAG{XSS_REFLECTED}
FLAG{XSS_STORED}
FLAG{XSS_DOM_BASED}
FLAG{WEAK_CREDENTIALS}
FLAG{BRUTE_FORCE_SUCCESS}
FLAG{SESSION_HIJACK}
FLAG{IDOR_USER_DATA}
FLAG{IDOR_PATH_TRAVERSAL}
FLAG{JWT_NONE_ALG_BYPASS}
FLAG{CMD_INJECTION_RCE}
FLAG{SSRF_METADATA_LEAK}
FLAG{XXE_FILE_READ}
FLAG{CSRF_EMAIL_TAKEOVER}
FLAG{LFI_PATH_TRAVERSAL}
FLAG{LFI_TO_RCE}
```

---

## ⚖ Aviso Legal Completo

Este proyecto es creado con **fines exclusivamente educativos** para la comunidad de ciberseguridad.

1. **Solo para uso en entornos controlados** — Propios o con autorización explícita por escrito.
2. **Aplicar estas técnicas sin permiso es ilegal** en la mayoría de jurisdicciones:
   - Colombia: Ley 1273 de 2009 (delitos informáticos) — hasta 8 años de prisión
   - España: Art. 197 bis y 264 del Código Penal
   - México: Art. 211 bis del Código Penal Federal
   - EE.UU.: Computer Fraud and Abuse Act (CFAA)
   - UE: Directiva NIS2 y legislación nacional
3. **Los datos son ficticios** — Cualquier similitud con datos reales es coincidencia.
4. **El autor no se responsabiliza** por el uso malintencionado de este material.

**El conocimiento de seguridad ofensiva es una herramienta. Úsala para defender, no para atacar.**

---

## 🤝 Contribuir

¿Quieres agregar módulos o mejorar el lab?

1. Fork del repositorio
2. Agrega tu módulo siguiendo el patrón existente
3. Incluye: teoría, laboratorio, pistas y flag
4. Pull request con descripción detallada

Ideas para nuevos módulos:
- Insecure Deserialization
- Race Conditions
- HTTP Request Smuggling
- GraphQL Injection
- NoSQL Injection
- SSTI (Server-Side Template Injection)
- OAuth 2.0 Misconfigurations

---

## 📖 Recursos Adicionales

### Plataformas de práctica gratuitas
- [HackTheBox](https://hackthebox.com) — Máquinas y challenges reales
- [TryHackMe](https://tryhackme.com) — Aprendizaje guiado paso a paso
- [PentesterLab](https://pentesterlab.com) — Web application security
- [PortSwigger Web Academy](https://portswigger.net/web-security) — El mejor recurso para web hacking (GRATIS)
- [DVWA](https://dvwa.co.uk) — Damn Vulnerable Web Application (local)
- [WebGoat (OWASP)](https://owasp.org/www-project-webgoat/) — App vulnerable de OWASP

### Documentación
- [OWASP Top 10 2021](https://owasp.org/Top10/)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [HackTricks](https://book.hacktricks.xyz) — Wiki de técnicas de hacking
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) — Payloads para todo

### Canales YouTube recomendados
- **IppSec** — Walkthroughs de HackTheBox
- **John Hammond** — CTFs y hacking educativo
- **LiveOverflow** — Seguridad profunda y técnica
- **NetworkChuck** — Hacking para principiantes

---

*HackLab Academy — Aprende. Practica. Defiende.* 🛡
