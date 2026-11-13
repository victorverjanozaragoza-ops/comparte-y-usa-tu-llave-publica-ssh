# Práctica P1.3: Comparte y usa tu llave SSH (GitHub y más allá de tu VM)

<p align="center">
  <a href="./practica-ssh-llave-publica-verjano.pdf" download>
    <img src="https://img.shields.io/badge/📥_Descargar_Práctica_en_PDF-red?style=for-the-badge&logo=adobeacrobatreader&logoColor=white" alt="Descargar PDF" height="40"/>
  </a>
</p>

<p align="center">
  <a href="./practica-ssh-llave-publica-verjano.pdf"><b>Descargar o ver la memoria completa en PDF</b></a>
</p>

---

* **Alumno:** Víctor Verjano
* **Curso:** 2º ASIR
* **Módulo:** Servicios de Red e Internet (SRI)
* **Práctica:** P1.3 — Comparte y usa tu llave SSH

---

## Objetivo de la práctica

Comprender que la misma pareja de claves SSH que utilizo en el aula (MADLinuX) para administrar la máquina virtual Debian (P101) o los contenedores en ValdeLAB36 (vlb36) es, en realidad, un pasaporte de autenticación digital universal. En esta práctica la configuro para acceder y trabajar de forma segura con **GitHub** mediante SSH sin necesidad de introducir usuario, contraseñas ni tokens personales (PAT).

---

## 0. Escenario y Criptografía Asimétrica

El esquema de trabajo se basa en un par de claves generado en mi equipo local (MADLinuX):

```text
  MADLinuX / Host (Local)                             Destinos Remotos
┌───────────────────────────────┐                   ┌─────────────────────────────────────────┐
│ Clave Privada:                │                   │ Servidores SSH / Repositorios           │
│ ~/.ssh/id_ed25519             │ ── NUNCA SALE ──► │ (NUNCA deben recibir la clave privada)  │
│                               │                   └─────────────────────────────────────────┘
│ Clave Pública:                │                   ┌─────────────────────────────────────────┐
│ ~/.ssh/id_ed25519.pub         │ ─── SE COPIA ───► │ 1. GitHub (Settings -> SSH keys)        │
│                               │                   │ 2. VM Debian (~/.ssh/authorized_keys)   │
│                               │                   │ 3. ValdeLAB36 (Servidor aldebaran)      │
└───────────────────────────────┘                   └─────────────────────────────────────────┘
```

> **Regla de oro de seguridad:** La clave privada (`id_ed25519`) nunca sale de mi ordenador y tiene permisos `600`. Únicamente se comparte y distribuye la clave pública (`id_ed25519.pub`).

---

## 1. Recuperación de la clave pública

En mi equipo compruebo la clave pública generada con el algoritmo de curvas elípticas **Ed25519**:

```bash
verjano@madlinux:~$ ssh-keygen -t ed25519 -C "verjano@madlinux"
Generating public/private ed25519 key pair.
Your identification has been saved in /home/verjano/.ssh/id_ed25519
Your public key has been saved in /home/verjano/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:Ee+cmpJLz6ApYkOD3UOpph26vjhcsT8OVU746xBEF/M verjano@madlinux

verjano@madlinux:~$ cat ~/.ssh/id_ed25519.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIImPfA5nJIHpYeBqQ7DkJgceA2ZES+O6ha8pq/gpVmxV verjano@madlinux
```

---

## 2. Añadir la clave a la cuenta de GitHub

Los pasos que he seguido en la interfaz web de GitHub:

1. Voy a mi foto de perfil (arriba a la derecha) → **Settings**.
2. En el menú de la izquierda elijo **SSH and GPG keys**.
3. Pulso en el botón verde **New SSH key**.
4. Pongo un título que me permita identificarla: `madlinux-aula`.
5. Tipo: `Authentication Key`.
6. En el campo *Key* pego la línea completa que me dio `cat ~/.ssh/id_ed25519.pub`.
7. Pulso en **Add SSH key**.

---

## 3. Ejercicio 1: Comprobación de la conexión con GitHub

Ejecuto la comprobación oficial con `ssh -T git@github.com` (con el parámetro `-T` para no pedir pseudo-terminal interactiva, ya que GitHub no ofrece shell remota):

```bash
verjano@madlinux:~$ ssh -T git@github.com
The authenticity of host 'github.com (140.82.121.4)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zPTO6fp539816SO79wTQ4.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Hi victorverjanozaragoza-ops! You've successfully authenticated, but GitHub does not provide shell access.
```

El saludo `Hi victorverjanozaragoza-ops! You've successfully authenticated...` confirma que GitHub ha verificado mi identidad mediante la clave pública registrada.

---

## 4. Ejercicio 2: Clonado de repositorio por SSH

Compruebo el funcionamiento clonando el repositorio mediante la URL de SSH (`git@github.com:...`) en vez de la URL HTTPS:

```bash
verjano@madlinux:~$ git clone git@github.com:victorverjanozaragoza-ops/comparte-y-usa-tu-llave-publica-ssh.git
Cloning into 'comparte-y-usa-tu-llave-publica-ssh'...
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (4/4), done.
```

El repositorio se descarga directamente sin pedir usuario ni contraseñas. A partir de ahora, cualquier comando `git push` o `git pull` se autenticará automáticamente en segundo plano.

---

## 5. Conecta los puntos: Una llave, muchas puertas

El patrón que hemos usado aquí es exactamente el mismo que en las prácticas anteriores:

| Dónde se ha usado | Qué guardamos y dónde | Para qué sirve |
| :--- | :--- | :--- |
| **VM Debian (P101)** | `id_ed25519.pub` en `~/.ssh/authorized_keys` de la VM | Acceder por terminal SSH sin teclear la contraseña |
| **ValdeLAB36 (RETO 1 vlb36)** | `id_ed25519.pub` en `aldebaran` mediante Incus | Administrar de forma segura los contenedores de prácticas |
| **GitHub (P1.3)** | `id_ed25519.pub` en Settings → SSH and GPG keys | Clonar, subir cambios (`push`) y gestionar repositorios |
| **Gitea / Forgejo / VPS propio** | `id_ed25519.pub` en la interfaz o en `authorized_keys` | Idéntico funcionamiento si autoalojamos un Git en ValdeLAB36 o en Nextcloud |

---

## 6. Ejercicios de Reflexión

### Ejercicio 3: ¿Qué pasaría si perdieras tu portátil y alguien copiara tu `~/.ssh/id_ed25519`? ¿Qué deberías hacer con esa llave en todos los sitios donde la instalaste (VM, vlb36, GitHub)?

**Impacto:**
Si alguien copia la clave privada (`id_ed25519`), tiene mi identidad digital completa. Podría entrar sin restricciones a mi máquina virtual Debian de clase, a mi contenedor de ValdeLAB36 y podría clonar, modificar o borrar repositorios en mi cuenta de GitHub con permisos de escritura.

**¿Por qué NO basta con cambiar la contraseña?**
Porque SSH por clave pública no consulta las contraseñas del sistema ni de GitHub. Funciona mediante un desafío criptográfico con la clave privada. Aunque cambie la contraseña de mi usuario de GitHub o de la máquina virtual, la clave privada seguirá siendo válida y el atacante podrá seguir accediendo.

**Procedimiento de revocación inmediata:**
1. **En GitHub:** Entrar a *Settings → SSH and GPG keys*, buscar la clave `madlinux-aula` y pulsar en **Delete** para revocarla.
2. **En la VM Debian:** Editar `~/.ssh/authorized_keys` y borrar la línea correspondiente a esa clave pública.
3. **En ValdeLAB36:** Eliminar la clave del fichero de claves autorizadas en el servidor `aldebaran` / Incus.
4. **Generar un nuevo par:** En el nuevo equipo seguro, generar un par nuevo con `ssh-keygen -t ed25519` (con una buena *passphrase*) y volver a repartir la nueva clave pública.

---

### Ejercicio 4: Explica con tus palabras por qué la misma llave te sirve para tu VM, para vlb36 y para GitHub. ¿Qué tienen en común los tres servidores?

Tienen en común que **todos ellos implementan el estándar del protocolo SSH (RFC 4252)** y la **criptografía asimétrica de clave pública y clave privada**.

A nivel de protocolo SSH, a los tres servidores les da igual el servicio que ofrezcan por encima (una shell interactiva en Debian, la gestión de un contenedor con Incus en ValdeLAB36 o la sincronización de objetos Git en GitHub). Todos ellos guardan nuestra clave pública y utilizan el mismo mecanismo de **reto-respuesta (challenge-response)**:
1. El servidor nos envía un reto cifrado aleatorio.
2. Nuestro cliente local lo firma con nuestra clave privada matemática.
3. El servidor comprueba la firma contra la clave pública que tiene registrada. Si coincide, valida la identidad y nos permite el paso sin enviar nunca información secreta por la red.

---

## 7. Archivos de la práctica

* [id_ed25519.pub](./id_ed25519.pub): Clave pública Ed25519 entregada.
* [practica-ssh-llave-publica-verjano.pdf](./practica-ssh-llave-publica-verjano.pdf): Memoria completa de la práctica generada en formato PDF.
