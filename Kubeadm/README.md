# En este caso estaremos usando VirtualBox para crear las maquinas virtuales
**Debemos tener instalado virtualbox, sino lo tienes, lo puede descargar**

- [Descargar VirtualBox](https://www.virtualbox.org/wiki/Downloads)

# Antes de que empieces a trabajar con Kubeadm, debes tener en cuenta lo siguiente

- Un host Linux compatible. El proyecto Kubernetes proporciona instrucciones genéricas para distribuciones de Linux basadas en Debian y Red Hat, y aquellas distribuciones sin un administrador de paquetes.
- 2 GB o más de RAM por máquina (menos dejará poco espacio para sus aplicaciones).
- 2 CPU o más.
- Conectividad de red completa entre todas las máquinas del clúster (la red pública o privada está bien).
- Nombre de host, dirección MAC y product_uuid únicos para cada nodo.
- Algunos puertos están abiertos en sus máquinas. Consulte [aquí](#verificar-los-puertos-requeridos) para obtener más detalles.
- Swap desactivado. Usted DEBE desactivar de intercambio para que el kubelet para que funcione correctamente.
- Tambien debemos tener Docker instalado en las maquinas que usaremos para el cluster.

## Verificar los puertos requeridos

| Protocol | Direction | Port Range | Porpose | Used By |
| -------- | --------- | ---------- | -------- | ------- |
| TCP | Inbound | 6443 | Kubernetes Api Server | All |
| -------- | --------- | ---------- | -------- | ------- |
| TCP | Inbound | 2379-2380 | etcd server client API | kube-apiserver,etcd|
| -------- | --------- | ---------- | -------- | ------- |
| TCP | Inbound | 10250 | Kubelet API | Self, Control plane |
| TCP | Inbound | 10251 | kube-scheduler | Self |
| -------- | --------- | ---------- | -------- | ------- |
| TCP | Inbound | 10252 | kube-controller-manager | Self |


# En este caso estaremos usando VirtualBox para crear las maquinas virtuales

**Debemos tener instalado virtualbox, sino lo tienes, lo puede descargar**

- [Descargar VirtualBox](https://www.virtualbox.org/wiki/Downloads)
