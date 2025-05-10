# Informe de Infraestructura AWS en Alta Disponibilidad con EC2 y NGINX

## Descripción General

Esta plantilla de AWS CloudFormation crea una infraestructura altamente disponible que incluye:

- Una VPC personalizada
- Dos subredes públicas en distintas zonas de disponibilidad
- Un gateway de Internet con ruta configurada
- Un grupo de seguridad que permite tráfico HTTP y SSH
- Un balanceador de carga (Load Balancer) en la capa de aplicación (ALB)
- Un grupo de Auto Scaling con dos instancias EC2 configuradas con NGINX

---

## Parámetros

| Nombre      | Descripción                                 | Tipo                          |
|-------------|---------------------------------------------|-------------------------------|
| `KeyName`   | Nombre del par de claves SSH existente      | `AWS::EC2::KeyPair::KeyName` |

---

## Recursos

### Red

- **VPC (`WebVPC`)**  
  Crea una red privada con DNS habilitado (`10.0.0.0/16`).

- **Subredes (`PublicSubnet1`, `PublicSubnet2`)**  
  Dos subredes públicas distribuidas en distintas zonas de disponibilidad (`10.0.1.0/24`, `10.0.2.0/24`).

- **Gateway de Internet (`InternetGateway`)**  
  Permite el acceso a Internet.

- **Asociaciones de rutas (`RouteTable`, `Route`, `SubnetRouteTableAssociationX`)**  
  Configura el enrutamiento para permitir tráfico externo hacia las subredes públicas.

### Seguridad

- **Grupo de seguridad (`WebSG`)**  
  Permite tráfico entrante en los puertos:
  - 22 (SSH)
  - 80 (HTTP)

### Cómputo

- **Plantilla de lanzamiento (`LaunchTemplate`)**  
  Define una instancia `t2.micro` con Amazon Linux 2 que instala y configura NGINX automáticamente.

- **Auto Scaling Group (`AutoScalingGroup`)**  
  Mantiene siempre dos instancias activas (min, max y desired en 2). Ambas instancias están registradas en un grupo objetivo del Load Balancer.

### Balanceador de Carga

- **Target Group (`TargetGroup`)**  
  Define el grupo de instancias detrás del Load Balancer, con chequeo de salud en `/`.

- **Load Balancer (`LoadBalancer`)**  
  Distribuye el tráfico HTTP entre las instancias EC2 en diferentes subredes.

- **Listener (`Listener`)**  
  Escucha en el puerto 80 y reenvía al grupo objetivo.

---

## Salidas

| Nombre             | Descripción                    |
|--------------------|--------------------------------|
| `LoadBalancerDNS`  | DNS público del Load Balancer  |


