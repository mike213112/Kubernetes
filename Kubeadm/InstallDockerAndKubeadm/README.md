## Instalar de Docker

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

**Agrear y Configurar el Demonio**

Antes de ejecutar el siguiente comando debemos tener en cuenta que tenemos que estar como superusuarios

**Ubuntu**
```bash
sudo su
```

**Debian**
```bash
su -
```

```bash
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
```

**Crear una carpeta para Docker Service**

```bash
mkdir -p /etc/systemd/system/docker.service.d
```

**Recargar el Demonio**

```bash
systemctl daemon-reload
```

**Reiniciar Docker**

```bash
systemctl restart docker
```


## Instalar Kubeamd

**Descargamos la clave de public Google Cloud**

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

**Agregamos el repositorio de Kubernetes**

```bash
apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

**Instalamos Kubeadm**

```bash
apt install -y kubeadm
```
