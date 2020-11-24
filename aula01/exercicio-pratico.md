<p align="center"><a href="../aula01">Aula 1</a></p>
<br/>

# Exercício prático - Aula 1

Para esta aula, usaremos nossa querida, estimada e valorosa aplicação `simplecrud`. Sim, a mesma dos exercícios anteriores. É necessário também que você tenha acesso a um cluster do OpenShift na versão 4.5 Você pode usar a Katacoda para criar cluster que duram uma hora, usar os labs no site da Red Hat ou até mesmo seguir nosso [apêndice sobre como instalar o CRC](../apendices/openshift_crc.md). Caso você possa investir num cluster para aprendizado, a licença do OpenShift é gratuita por 60 dias e você pode hospedá-lo na Azura, AWS, IBM Cloud ou na Red Hat, mas a infraestruturas e as máquinas onde o OpenShift será hospedado não são gratuitas, e os recursos necessários são vastos. Para mais detalhes, veja a [seção de teste do OpenShift](https://www.openshift.com/try).

## Requerimentos
- Você deve ter acesso de desenvolvedor a um cluster do OpenShift
- Será necessário ter uma conta no Quay.io
- Seu projeto deverá se chamar capitulo03
- Sua aplicação deverá se chamar aula01
- Sua aplicação deverá ser criada com uma configuração de implantação
- A imagem da aplicação deverá estar hospedada no Quay.io
- A imagem da aplicação deverá ser chamada simplecrud, com a tag aula01
- Seu Dockerfile deverá clonar o código contido em https://github.com/mentoria-openshift/simplecrud-spring.git
- O objeto JSON para teste da aplicação estará disponível do endpoint `/api`

## Passo-a-passo
Este passo a passo indica como chegar ao resultado final do exercício. Clique na seta ao lado do enunciado para exibir a resposta. Note que o exercício todo será feito na linha de comando.

<details> 
  <summary>1. Use como base o Dockerfile construído no capítulo 1, na aula 2</summary>
  
Dockerfile
```Dockerfile
FROM docker.io/maven:3.6.3-adoptopenjdk-11

ENV APP_PROFILE="default"

COPY [ "scripts/entrypoint.sh", "/entrypoint.sh" ]

RUN apt update -y
RUN apt install -y git
RUN git clone https://github.com/mentoria-openshift/simplecrud-spring /opt/simplecrud

WORKDIR /opt/simplecrud

RUN mvn clean install

EXPOSE 8080
ENTRYPOINT [ "sh", "/entrypoint.sh" ]
```

scripts/entrypoint.sh
```bash
#!/bin/bash

java -Dspring.profiles.active="$APP_PROFILE" -jar /opt/simplecrud/target/simplecrud-0.0.1-SNAPSHOT.jar 
```

</details>

<details> 
  <summary>2. Faça as correções necessárias de permissões e camadas para o OpenShift </summary>

Dockerfile
```Dockerfile
FROM docker.io/maven:3.6.3-adoptopenjdk-11

LABEL maintainer="Seu Nome <voce@dominio.com>" \
  io.openshift.tags="java,spring,h2" \
  io.k8s.description="Exercício para o capítulo 3" \
  io.openshift.expose-services="8080:http" \
  openshift.io/display-name="Simple Java CRUD"

ENV APP_PROFILE="default"

COPY [ "scripts/entrypoint.sh", "/entrypoint.sh" ]

RUN apt update -y && \
  apt install -y git && \
  git clone https://github.com/mentoria-openshift/simplecrud-spring /opt/simplecrud

WORKDIR /opt/simplecrud

RUN mvn clean install && \
  chown -R 1001:0 /opt/simplecrud && \
  chmod g=u -R /opt/simplecrud

USER 1001
EXPOSE 8080
ENTRYPOINT [ "sh", "/entrypoint.sh" ]
```

</details>

<details> 
  <summary>3. Compile a imagem e copie-a para o Quay.io</summary>

```bash
# Com podman e skopeo
podman build -t simplecrud:aula01 .
skopeo login -u seu_usuario quay.io
skopeo copy containers-storage:localhost/simplecrud:aula01 docker://quay.io/seu_usuario/simplecrud:aula01

# Com docker
docker build -t simplecrud:aula01 .
docker login -u seu_usuario quay.io
docker push quay.io/seu_usuario/simplecrud:aula01
```

</details>

<details> 
  <summary>4. Faça login no OpenShift e crie o projeto</summary>
  
```bash
oc login -u seu_usuario https://api.cluster.com

oc new-project capitulo03
```

</details>

<details> 
  <summary>5. Crie a aplicação usando como base o endereço da imagem</summary>
   
```bash
oc new-app --name aula01 --as-deployment-config --docker-image quay.io/seu_usuario/simplecrud:aula01
```

</details>

<details> 
  <summary>6. Exponha o serviço da aplicação numa rota externa</summary>
   
```bash
oc expose svc aula01

oc get route
```

</details>

<details> 
  <summary>7. Teste a aplicação</summary>

```bash
curl http://aula01-capitulo03.apps.cluster.com/api
```

</details>

---
<p align="center"><a href="../aula01">Aula 1</a></p>