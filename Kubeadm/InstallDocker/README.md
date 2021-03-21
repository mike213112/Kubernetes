## Instalacion de docker

**Primero debemos actualizar nuestro sistema operativo**

```bash
sudo apt-get update
```

**Instalar Https, Certificados, Curl**

```bash
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

**Debemos agregar la clabe GPG oficial de Docker**

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

**Utilizaremos el siguiente comando para configurar el repositorio stable**

```bash
echo \
"deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
**Instalaremos Docker Engine**

```bash
sudo apt update; sudo apt-get install docker-ce docker-ce-cli containerd.io
```

**Para utilizar Docker como usuario no root, utilizaremos el siguiente comando**

```bash
sudo usermod -aG docker $USER
```
