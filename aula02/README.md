<p align="center"><a href="../aula01">❮ Aula anterior</a> | <a href="../aula03">Próxima aula ❯</a></p>
<br/>

# Aula 2 - A estratégia Source-to-Image (S2I)
Bem-vindo à terceira aula do curso prático de OpenShift. Neste capítulo falaremos um pouco sobre a estratégia Source-to-Image do OpenShift, que permite que aplicações sejam compiladas e implantadas diretamente do código fonte. 

## Funcionamento do S2I
A estratégia S2I permite que aplicações sejam compiladas e implantadas diretamente do código fonte, a partir de um repositório git onde os arquivos estão hospedados. Ao final do processo, uma imagem é gerada contendo a aplicação compilada e pronta para ser executada. Conforme vimos anteriormente, o OpenShift permite uma sintaxe simples para o comando `oc new-app`, podendo ser fornecido apenas o URL do repositório Git, e uma verificação é feita para saber qual estratégia usar. Mas, apesar disso, esta lógica não é totalmente perfeita, então para prevenir erros, é ideal fornecer flags explícitos da estratégia utilizada. 

Em ambos os casos, quando a estratégia selecionada é o S2I, o OpenShift faz uma outra verificação para determinar a linguagem utilizada caso ela não seja fornecida. Determinados arquivos e extensões contidas no seu código fonte indicam a linguagem específica na qual sua aplicação foi escrita, e o OpenShift usa esses arquivos para determinar qual imagem de compilação usar. Esta ferramenta é dependente de arquivos específicos. Aqui está uma lista dos formatos mais comuns usados no OpenShift, apesar de a lista completa suportada pela plataforma ser muito maior.

<center>

|Arquivo|Imagem de compilação|Linguagem|
|---|---|---|
|`pom.xml`|jee|Java (com JBoss)|
|`package.json`|nodejs|Node.js|
|`index.php`|php|PHP|
|`index.html`|httpd|HTML|

</center>

Esta detecção, no entanto, também não é perfeita. Diferentes versões de uma mesma linguagem usam a mesma estrutura e os mesmos arquivos, e isso pode confundir o OpenShift pelo fato de a busca ficar ambígua. Por exemplo, ter o arquivo `index.php` no seu projeto indica que ele deve ser compilado como PHP, mas não especifica a versão. O OpenShift pode subir a aplicação com o PHP 8.0 apesar de ela ter sido escrita com PHP 5.5, por exemplo. Para evitar esse tipo de ambiguidade, o ideal é ser explicito sobre qual imagem de compilação usar no seu projeto.

O OCP fornece duas formas de criar uma aplicação S2I com uma versão específica de linguagem: uma notação simplificada e outra com flag de fluxo de imagem. A diferença entre as duas formas é que a primeira é específica para S2I, e usa o til (~) para separar imagem do repositório, enquanto a segunda opção é a mesma usada para implantar aplicações a partir de um fluxo de imagem, como padrão. O segundo formato simplesmente implanta uma imagem quando não existe um repositório fornecido, e quando um é fornecido o OCP entende que se trata de uma compilação S2I.

```bash
# Criando uma aplicação com S2I sem especificar versões ou estratégias
oc new-app --name minha_aplicacao --as-deployment-config https://github.com/seu_usuario/aplicacao_php.git

# Criando uma aplicação S2I especificando versões (sem notação)
oc new-app --name minha_aplicacao --as-deployment-config -i php:8 https://github.com/seu_usuario/aplicacao_php.git

# Criando uma aplicação S2I especificando versões (com notação)
oc new-app --name minha_aplicacao --as-deployment-config php:8~https://github.com/seu_usuario/aplicacao_php.git
```

### O processo de compilação
A estratégia S2I depende de scripts específicos para funcionar, cada um com uma função. Compilação da aplicação, execução de testes, execução da aplicação em si, manual, compilações incrementais. Enfim, é uma solução completa e avançada, que precisa de cuidados específicos. Alguns desses scripts são obrigatórios, enquanto outros não.

<center>

|Script|Obrigatório|Descrição
|---|---|---|
|`assemble`|sim|O script `assemble` compila a aplicação e a coloca no diretório apropriado dentro da imagem.|
|`run`|sim|O script `run` executa a aplicação. É recomendado o uso do comando `exec` ao executar processos de container para assegurar desligamento gracioso dos processos iniciados.|
|`save-artifacts`|não|Este script salva as dependências baixadas para que elas sejam usadas nos próximos passos de uma compilação incremental. Isso tira a necessidade de baixar novamente as dependências.|
|`usage`|não|Uma documentação simples sobre como usar seu script e o que ele faz.|
|`test/run`|não|Semelhante ao script `run`, mas colocado na pasta `test`. Serve para executar testes da sua aplicação para assegurar que tudo funciona.|

</center>

Quando uma compilação S2I é iniciada, um arquivo tar é criado contendo o código fonte da aplicação e os scripts S2I. O arquivo tar é, então, extraído, e seu conteúdo é colocado na pasta especificada na imagem S2I, na label `io.openshift.s2i.destination`. O caminho padrão é `/tmp`.

No caso de ser uma compilação incremental, o script `assemble` é executado, e a aplicação é, então, compilada. O script `assemble`, ao terminar de compilar a imagem, coloca os arquivos binários da aplicação no diretório apropriado, e, no caso de ser uma compilação incremental, o script `save-artifacts` entra em cena e salva os artefatos apropriadamente, num arquivo tar, e o próximo passo da compilação é executado e os demais scripts chamados, até que esse processo de compilação finaliza, e o script `run` sobe a aplicação.

### Criando uma imagem de compilação
#### Definição da imagem
Uma imagem de compilação é criada como qualquer outra: a partir de um Dockerfile. Mas temos que fornecer algumas informações específicas, como labels. Assim como com imagens comuns para o OpenShift, cada label tem sua função no S2I. No geral, as mesmas labels são usadas, mas com uma adição: `io.openshift.s2i.scripts-url`. 

Esta label é imprescindível para imagens S2I, pois é ela que define onde os scripts de compilação e execução estarão localizados. Além de definir o camindo dos scripts, é necessário copiá-los para a pasta definida na label.

```Dockerfile
FROM registry.access.redhat.com/ubi8/ubi:8.0

LABEL maintainer="Seu Nome <voce@email.com>" \
    io.k8s.description="Uma imagem para compilação de aplicações Maven escritas com o OpenJDK 11" \
    io.k8s.display-name="OpenJDK 11 S2I Builder" \
    io.openshift.expose-services="8080:http" \
    io.openshift.tags="java,openjdk,maven,s2i,builder,jdk,spring,vertx,vert.x,wildfly" \
    io.openshift.s2i.scripts-url="/usr/libexec/s2i"

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

COPY ./s2i/bin/ /usr/libexec/s2i

USER 1001
EXPOSE 8080
CMD ["/usr/libexec/s2i/usage"]
```
#### Os scripts S2I
Além de definir o Dockerfile, precisamos definir os scripts mencionados acima, trabalhando primariamente com os obrigatórios, que são `run` e `assemble`. 

```bash
#!/bin/bash -e

# Caso o script seja executado com -h, instruções são exibidas
if [[ "$1" == "-h" ]]; then
	exec /usr/libexec/s2i/usage
fi

# Restaura dependências do maven para a pasta do .m2
if [ "$(ls /tmp/artifacts/ 2>/dev/null)" ]; then
  echo "--> Restaurando artefatos de compilação..."
  shopt -s dotglob
  mv /tmp/maven-artifacts /.m2
  shopt -u dotglob
fi

# Copia o código fonte para sua pasta
echo "--> Instalando código fonte da aplicação..."
cp -Rf /tmp/src/. /opt/java-app

# Executa os testes
echo "--> Executando testes unitários da aplicação..."
mvn test -f /opt/java-app/pom.xml

# Compila a aplicação
echo "--> Compilando código fonte da aplicação..."
mvn clean install -DskipTests -f /opt/java-app/pom.xml
```

E o script de execução
```bash
#!/bin/bash -e

# Inicia a execução da aplicação
echo "--> Iniciando aplicação a partir de arquivo JAR"
exec java -jar /opt/java-app/target/*.jar
```

Isso é o suficiente para que sua imagem fique pronta. Para subi-la para o OpenShift, é possível usar ambos dos métodos ensinados nas aulas anteriores: importando a imagem para o OpenShift ou até mesmo compilando o Dockerfile diretamente.

### Modificando scripts de uma imagem já existente
Além de criar uma imagem do zero com o que precisamos, podemos reaproveitar uma imagem já existente. Alterando os scripts de execução e compilação, usaremos apenas as dependências da imagem original. Isso é feito criando os scripts modificados diretamente na sua aplicação, no diretório `.s2i/bin`, e ao buscar o código, o OpenShift verá os arquivos definidos e usará eles ao invés dos arquivos padrão da imagem.

Usando as definições acima, suponhamos que você já tenha aquela imagem compilada no seu OpenShift e precisa alterar o funcionamento para uma aplicação em específico. O goal `generate-sources` do Maven precisa ser executado após os goals de compilação e antes dos de testes unitários.

Será necessário criar, na raiz da aplicação, os diretórios `.s2i/bin`, e colocar o script `assemble` ali dentro. Um exemplo do script novo:

```bash
#!/bin/bash -e

# Caso o script seja executado com -h, instruções são exibidas
if [[ "$1" == "-h" ]]; then
	exec /usr/libexec/s2i/usage
fi

# Restaura dependências do maven para a pasta do .m2
if [ "$(ls /tmp/artifacts/ 2>/dev/null)" ]; then
  echo "--> Restaurando artefatos de compilação..."
  shopt -s dotglob
  mv /tmp/maven-artifacts /.m2
  shopt -u dotglob
fi

# Copia o código fonte para sua pasta
echo "--> Instalando código fonte da aplicação..."
cp -Rf /tmp/src/. /opt/java-app

# Compila a aplicação
echo "--> Compilando código fonte da aplicação..."
mvn clean install -DskipTests -f /opt/java-app/pom.xml

# Gera arquivos fonte
echo "--> Gerando arquivos de código fonte..."
mvn generate-sources -f /opt/java-app/pom.xml

# Executa os testes
echo "--> Executando testes unitários da aplicação..."
mvn test -f /opt/java-app/pom.xml
```

Jogando esse script na pasta definida, o OpenShift executará ele ao invés do script contido na imagem.

## Referências
* [Documentação do OpenShift](https://docs.openshift.com/)
* [Estratégias de compilação](https://docs.openshift.com/container-platform/4.5/builds/build-strategies.html)
* [Entendendo compilação de imagens](https://docs.openshift.com/container-platform/4.5/builds/understanding-image-builds.html)
* [Resolução de problemas S2I](https://docs.openshift.com/container-platform/4.5/support/troubleshooting/troubleshooting-s2i.html)
* [Criando imagens](https://docs.openshift.com/container-platform/4.5/openshift_images/create-images.html)

----
<p align="center"><a href="../aula01">❮ Aula anterior</a> | <a href="../aula03">Próxima aula ❯</a></p>