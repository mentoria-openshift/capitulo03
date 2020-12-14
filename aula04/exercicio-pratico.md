<p align="center"><a href="../aula04">Aula 4</a>
<br/>

# Exercício prático - Aula 4

Este exercício colocará em prática as coisas ensinadas na aula 4. Aqui você criará uma imagem S2I e fará a implantação dela a partir de seu código fonte. Ao término desta aula, você será capaz de entender os conceitos do S2I e como criar uma imagem.

## Requerimentos
- Você deve ter acesso de desenvolvedor a um cluster do OpenShift
- Será necessário ter uma conta no GitHub
- Será necessário ter uma conta no Quay.io
- Sua imagem deverá ser hospedada com nome `quay.io/seu_usuario/openjdk-aula04:1.0`
- Seu projeto deverá se chamar capitulo03
- Sua aplicação deverá se chamar aula04
- Sua aplicação deverá ser criada com uma configuração de implantação
- O código da aplicação deverá estar hospedado no GitHub (forkado do simplecrud do curso para sua conta)
- O objeto JSON para teste da aplicação estará disponível do endpoint `/api`
- Crie um fluxo de imagem chamado `openjdk-aula04` com a tag `1.0` para hospedar sua imagem de compilação.
- Você deverá commitar suas alterações de scripts na branch `s2i-criacao`.
- Execute 2 passos de compilação: `mvn clean install -DskipTests` para compilar e `mvn test` para testes unitários.
- Armazene os scripts de compilação na pasta `/opt/s2i-scripts`.
- Sua imagem de compilação S2I deverá usar a imagem `registry.access.redhat.com/ubi8/ubi:8.0` como base
- Ela deverá redirecionar o tráfego da porta 8080 (exposta pelo Spring Boot) para a porta 80 (que será usada pelo serviço do OpenShift)

## Passo-a-passo
Este passo a passo indica como chegar ao resultado final do exercício. Clique na seta ao lado do enunciado para exibir a resposta. Note que o exercício todo será feito na linha de comando.

<details> 
  <summary>1. Crie uma nova imagem de compilação com a ferramenta S2I</summary>
  
```bash
s2i create openjdk-aula04 openjdk-aula04
```

</details>

<details> 
  <summary>2. Corrija o Dockerfile. Inclua as labels necessárias e as instruções para OpenJDK e Maven, além da correção de permissões e cópia dos scripts.</summary>
  
Assemble:
```Dockerfile
FROM registry.access.redhat.com/ubi8/ubi:8.0

LABEL maintainer="Seu Nome <voce@email.com>" \
    io.k8s.description="Imagem de compilação do capítulo 3" \
    io.k8s.display-name="OpenJDK 11 S2I Builder" \
    io.openshift.expose-services="8080:http" \
    io.openshift.tags="java,openjdk,maven,s2i,builder,jdk" \
    io.openshift.s2i.scripts-url="/opt/s2i-scripts"

ENV JAVA_HOME="/opt/openjdk11" \
    MAVEN_HOME="/opt/maven" \
    PATH="$PATH:/opt/maven/bin:/opt/openjdk11/bin"

WORKDIR /opt
RUN yum update -y && \
    yum install -y wget curl tar && \
    wget https://download.java.net/java/GA/jdk11/9/GPL/openjdk-11.0.2_linux-x64_bin.tar.gz && \
    wget https://downloads.apache.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz && \
    tar xvzf openjdk-11.0.2_linux-x64_bin.tar.gz && \
    tar xvzf apache-maven-3.6.3-bin.tar.gz && \
    mv apache-maven-3.6.3 /opt/maven && \
    mv jdk-11.0.2 /opt/openjdk11 && \
    mkdir /.m2 && \
    chown -R 1001:0 /opt /.m2 && \
    chmod -R g=u /opt /.m2

COPY ./s2i/bin/ /opt/s2i-scripts

USER 1001
EXPOSE 8080
CMD ["/opt/s2i-scripts/usage"]
```

</details>

<details> 
  <summary>3. Modifique os scripts `run` e `assemble`. Os demais não são necessários.</summary>
  
Assemble:
```bash
#!/bin/bash

# Caso o script seja executado com -h, instruções são exibidas
if [[ "$1" == "-h" ]]; then
	exec /usr/libexec/s2i/usage
fi

# Restaura dependências do maven para a pasta do .m2
if [ "$(ls /tmp/artifacts/ 2>/dev/null)" ]; then
  echo "--> Restaurando artefatos de compilação..."
  shopt -s dotglob
  mv /tmp/artifacts ${HOME}/.m2
  shopt -u dotglob
fi

# Copia o código fonte para sua pasta
echo "--> Instalando código fonte da aplicação..."
cp -Rf /tmp/src/. /opt/java-app

# Compila a aplicação
echo "--> Compilando código fonte da aplicação..."
mvn clean install -DskipTests -f /opt/java-app/pom.xml

# Executa os testes
echo "--> Executando testes unitários da aplicação..."
mvn test -f /opt/java-app/pom.xml
```

Run:
```bash
#!/bin/bash

# Inicia a execução da aplicação
echo "--> Iniciando aplicação a partir de arquivo JAR"
exec java -jar /opt/java-app/target/*.jar
```

</details>

<details> 
  <summary>4. Suba a aplicação para o Quay.io</summary>
  
```bash
# Comando deve ser executado na pasta do Dockerfile
podman build -t openjdk-aula04:1.0 .

# Copiando para o repositório
skopeo login -u seu_usuario quay.io
skopeo copy containers-storage:openjdk-aula04:1.0 quay.io/seu_usuario/openjdk-aula04:1.0
```

</details>

<details> 
  <summary>5. Importe a imagem para o OpenShift e crie sua aplicação</summary>
  
```bash
# Criação de segredo para autenticação
oc create secret docker-registry quayio \
  --docker-username=seu_usuario \
  --docker-password=sua_senha \
  --docker-server=quay.io

oc secrets link default quayio --for pull

# Importação da imagem
oc import-image openjdk-aula04:1.0 --from quay.io/seu_usuario/openjdk-aula04:1.0 --confirm

# Criação da aplicação
oc new-app --name aula04 openjdk-aula04:1.0~https://github.com/seu_usuario/simplecrud-spring.git#s2i-criacao
```

</details>

<details> 
  <summary>6. Exponha a aplicação e teste-a</summary>
  
```bash
oc expose svc aula04
oc get route

curl aula04-capitulo03.apps.cluster.com/api
{"hostname":"simplecrud-1-f7zn2","Mensagem a todos":"Mas para onde vai o mundo?","Banco de dados":"H2","Perfil da aplicação":"default","OpenShift é muito":"maneiro","Mensagem":"Olá, Mundo!","Está aplicação está rodando em":"português"}
```

</details>

---
<p align="center"><a href="../aula04">Aula 4</a>