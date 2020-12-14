<p align="center"><a href="../aula02">❮ Aula anterior</a> | <a href="../aula04">Próxima aula ❯</a></p>
<br/>

# Aula 3 - Modificando uma imagem de compilação S2I
Nesta aula, usaremos os conceitos aprendidos na aula anterior para modificar uma imagem existente do Source-to-Image. Também exploraremos alguns comandos novos do OpenShift, para buscar itens e descreve-los, além de como criar uma aplicação usando o S2I.

## Informações das imagens
Os scripts são empacotados nas imagens S2I por padrão, e podemos sobrescreve-los usando scripts próprios contidos na nossa aplicação. Em alguns cenários isso é aplicável, pois uma imagem já existente no OpenShift pode já ter as dependências necessárias para que nossa aplicação seja compilada e executada, mas as instruções dos scripts não são exatamente ideiais. Portanto, podemos mudar a forma como a aplicação é entregue sem criar uma nova imagem S2I.

Claro, dependendo da quantidade de customização necessária, talvez seja mais prudente criar uma nova imagem S2I do zero. Aprenderemos como fazer isso na próxima aula. Por ora, vamos alterar uma já existente.

O primeiro passo é buscar um fluxo de imagem já existente. Usamos os comandos `get` e `describe` para buscar informações de recursos no OpenShift, o que inclui fluxos de imagem. 

```bash
# Buscando o fluxo de imagem do PHP
oc get is -n openshift php

NAME   IMAGE REPOSITORY                                                                  TAGS             UPDATED
php    default-route-openshift-image-registry.apps.na45.prod.nextcle.com/openshift/php   7.2,7.3,latest   3 months ago
```

Note o switch `-n`, tendo como parâmetro `openshift`. Isso indica o projeto, que em comandos CLI é tido como _namespace_, e por seu switch é `-n`, podendo também ser alongado para `--namespace`. Isso se dá pois o fluxo de imagem que vamos buscar está contido no projeto `openshift`, já existente por padrão no OCP. Caso o flag de namespace seja omitido, o OpenShift buscará pelo recurso solicitado dentro do projeto em uso. Portanto, caso você esteja no projeto A e precisa de um recurso de um outro projeto, deverá usar `oc get <recurso> -n <nome do projeto do recurso>`. Este flag não resume somente ao comando `get`, e existe para diversos outros comandos do OpenShift.

Tendo o nome do fluxo de imagem, podemos descrevê-lo para saber o que existe dentro dele. Para isso, usamos o comando `describe`. Isso mostra todas as informações do fluxo de imagem fornecido, como tags contidas, nome do fluxo de imagem, registro de cada imagem de cada tag, número de hash das imagens tagueadas. E, novamente, o comando `describe` não serve somente para fluxos de imagem: ele serve para qualquer recurso do OpenShift, e ao usá-lo, você terá como retorno as informações gerais do recurso solicitado.

```bash
# Buscando informações
oc describe is php -n openshift

Name:			php
Namespace:		openshift
Created:		3 months ago
Labels:			samples.operator.openshift.io/managed=true
Annotations:		openshift.io/display-name=PHP
			openshift.io/image.dockerRepositoryCheck=2020-08-21T04:11:16Z
			samples.operator.openshift.io/version=4.5.6
Image Repository:	default-route-openshift-image-registry.apps.na45.prod.nextcle.com/openshift/php
Image Lookup:		local=false
Unique Images:		2
Tags:			3

7.3 (latest)
  tagged from registry.redhat.io/rhscl/php-73-rhel7:latest
    prefer registry pullthrough when referencing this tag

  Build and run PHP 7.3 applications on RHEL 7. For more information about using this builder image, including OpenShift considerations, see https://github.com/sclorg/s2i-php-container/blob/master/7.3/README.md.
  Tags: builder, php
  Supports: php:7.3, php
  Example Repo: https://github.com/sclorg/cakephp-ex.git

  * registry.redhat.io/rhscl/php-73-rhel7@sha256:962d936b00f4953d977be9b85268a289bf2f9d49df4d51b408a7c7835a551103
      3 months ago

7.2
  tagged from registry.redhat.io/rhscl/php-72-rhel7:latest
    prefer registry pullthrough when referencing this tag

  Build and run PHP 7.2 applications on RHEL 7. For more information about using this builder image, including OpenShift considerations, see https://github.com/sclorg/s2i-php-container/blob/master/7.2/README.md.
  Tags: builder, php
  Supports: php:7.2, php
  Example Repo: https://github.com/sclorg/cakephp-ex.git

  * registry.redhat.io/rhscl/php-72-rhel7@sha256:feb82582395558abc7823cff3800753aaa7bf976c5a4869bc921dd53e7eef447
      3 months ago
```

Note o trecho que diz "tagged from" de cada tag contida neste fluxo de imagem. Este atributo mostra o registro da imagem, o nome dela e a tag da imagem original. Precisamos deste endereço para buscar a imagem.

## Verificando os scripts originais
Os scripts modificados normalmente serão baseados nos originais, então baixaremos a imagem e verificaremos o que existe lá dentro. Para desenvolvedores mais experientes, criar um script do zero sem se basear nos originais pode ser vatanjoso, entretanto. 

Com o endereço da imagem original em mãos, podemos fazer um _pull_ dela. Vamos usar a versão 7.3 no exemplo.

```
podman pull registry.redhat.io/rhscl/php-73-rhel7:latest
```

Com a imagem em mãos, precisamos verificar o caminho dos scripts. Lembra-se da label mencionada na aula anterior? Ela é um pedaço chave neste trecho. O comando `inspect` do OpenShift foi baseado no mesmo comando do Docker e do Podman. Agora, usaremos o comando `inspect` para ver detalhes da nossa imagem, e buscar pela label especificamente.

```
podman inspect --format='{{ index .Config.Labels "io.openshift.s2i.scripts-url"}}' rhscl/php-73-rhel7

image:///usr/libexec/s2i
```

Neste ponto, podemos prosseguir de três formas: 

1. Criar um novo script assemble para nossa aplicação que chama o original. Ou seja, os passos do script original serão executados com adição de passos customizados. 
2. Fazer o nosso script sem mencionar o original, e, desta forma, o original não é executado.
3. Copiar o código do script original e colocá-lo no nosso. Assim as instruções do script original continuam sendo executados, mas sem chamá-lo.

Para as opções 1 e 2 não precisaremos do código do script original, mas para a 3 sim. Nos dois primeiros casos, basta criar os scripts e colocá-los na pasta correta, e depois subi-los com o S2I. 

Ao criarmos nossos scripts, eles deverão ser colocados na pasta `.s2i/bin`, na mesma pasta onde o arquivo de linguagem se encontra (seu `pom.xml`, `index.php` ou qualquer outro).

### Criando como base no script original
Para isso, precisaremos subir a imagem num container e imprimir o código do script. Vamos subir um container com a imagem.

```bash
podman run --name s2i-php-73 -it registry.redhat.io/rhscl/php-73-rhel7:latest bash
```

Isso subirá o container já com o bash aberto. Desta forma, uma vez que saímos do bash, o container morre. Vamos buscar o conteúdo dos scripts originais.

```bash
# Buscando o script de compilação
bash-4.2$ cat /usr/libexec/s2i/assemble

#!/bin/bash

set -e

source ${PHP_CONTAINER_SCRIPTS_PATH}/common.sh

shopt -s dotglob
echo "---> Installing application source..."
rm -fR /tmp/src/.git
mv /tmp/src/* ./

# Fix source directory permissions
fix-permissions ./
fix-permissions ${HTTPD_CONFIGURATION_PATH}

# Change the npm registry mirror if provided
if [ -n "$NPM_MIRROR" ]; then
        npm config set registry $NPM_MIRROR
fi
................................................
```

Para usar as mesmas instruções, temos que copiar todo o conteúdo desse script e colar no nosso. Mas, novamente, caso a intenção seja executar os scripts originais, a escolha mais prudente seria simplesmente chamá-los nos nossos scripts customizados.

### Criando scripts novos
Precisamos criar os diretórios e os scripts no caminho mencionado no tópico anterior. Como mencionado na aula anterior, as imagens OpenJDK padrão do OpenShift compilam aplicações com JBoss, o que a torna incompatível com aplicações que usem Spring Boot, por exemplo. Mas como as dependências são parecidas, e a imagem padrão tem OpenJDk e Maven instalados, podemos utiliza-la, apenas mudando a lógica de como a aplicação é compilada e servida. Vamos explorar o script original.

```bash
# Buscando o fluxo de imagem
oc get is --namespace openshift java
oc describe is --namespace openshift openjdk-11-rhel8
...................
tagged from registry.redhat.io/openjdk/openjdk-11-rhel8:1.0
...................

# Baixando a imagem
podman pull registry.redhat.io/openjdk/openjdk-11-rhel8:1.0

# Inspecionando a imagem
podman inspect --format='{{ index .Config.Labels "io.openshift.s2i.scripts-url"}}' registry.redhat.io/openjdk/openjdk-11-rhel8:1.0

image:///usr/local/s2i

# Subindo container
podman run --name s2i-jdk-11 -it registry.redhat.io/openjdk/openjdk-11-rhel8:1.0 bash

# Buscando arquivos
[jboss@28690d2dd508 ~]$ cat /usr/local/s2i/assemble
[jboss@28690d2dd508 ~]$ cat /usr/local/s2i/run 
```

Examine o conteúdo dos scripts de compilação e de execução. Note que existem linhas específicas. Eles não servem para uma aplicação comum, que usa apenas maven e executa diretamente do jar compilado.

Vamos criar um arquivo que faz uma compilação maven simples, sem execução de testes unitários. Esse será o valor do `assemble`.

```bash
#!/bin/bash

# Caso o script seja executado com -h, instruções são exibidas
if [[ "$1" == "-h" ]]; then
	exec /usr/libexec/s2i/usage
fi

# Copia o código fonte para sua pasta
echo "--> Instalando código fonte da aplicação..."
cp -Rf /tmp/src/. /opt/java-app

# Compila a aplicação
echo "--> Compilando código fonte da aplicação..."
mvn clean install -DskipTests -f /opt/java-app/pom.xml
```

Conteúdo do `run`:

```bash
#!/bin/bash

# Inicia a execução da aplicação
echo "--> Iniciando aplicação"
exec java -jar /opt/java-app/target/*.jar
```

Ao commitarmos esta alteração para o repositório da imagem e criarmos uma aplicação S2I a partir do código fonte, o OpenShift executará esses dois scripts ao invés dos padrões. Com isso, a imagem S2I padrão do OpenShift, feita para JBoss, se torna compatível com aplicações maven simples e Spring Boot.

## Compilações incrementais
É comum utilizar ferramentas de CI/CD integradas com o OpenShift, como Jenkins, TravisCI ou até mesmo ferramentas de pipeline da AWS, Azure e IBM Cloud. Nessa técnica, a aplicação é compilada e implantada diversas vezes, cada passo executando diferentes comandos e sem nenhuma intervenção manual. 

Como containers são de natureza imutável, esses passos podem ser difíceis, e a compilação da aplicação é dependente de determinadas bibliotecas. Usando o S2I, o desenvolvedor tem à sua disponsição mecanismos que permitem que essas dependências sejam reutilizadas nos diferentes passos, a fim de economizar tempo não baixando essas dependências novamente.

O script `save-artifacts` é executado depois do `assemble` para que essas dependências sejam salvas num local compartilhado, e no passo seguinte o script `assemble` busca os arquivos na pasta usada para salvar. Dependências Maven, por exemplo, que são salvos na pasta `.m2`, podem ser copiadas para a área compartilhada, e posteriormente buscada pelo passo de `assemble` para serem reutilizadas.

Exemplo do `save-artifacts`:

```bash
#!/bin/sh -e

# Salvamento da pasta .m2 com os artefatos
if [ -d ${HOME}/.m2 ]; then
    pushd ${HOME} > /dev/null
    tar cf - .m2
    popd > /dev/null
fi
```

Trecho do `assemble` para restaurar os arquivos:
```bash
# Restauração da pasta maven
...
if [ -d /tmp/artifacts/.m2 ]; then
  echo "---> Restaurando artefatos Maven..."
  mv /tmp/artifacts/.m2 ${HOME}/
fi
...
```

## Exercícios
Isso conclui a aula de modificação de imagens S2I no OpenShift. Para praticar o conteúdo aprendido, vamos fazer um exercício prático.

* [Questionário](questionario.md)
* [Exercício prático](exercicio-pratico.md)

## Referências
* [Documentação do OpenShift](https://docs.openshift.com/)
* [Estratégias de compilação](https://docs.openshift.com/container-platform/4.5/builds/build-strategies.html)
* [Entendendo compilação de imagens](https://docs.openshift.com/container-platform/4.5/builds/understanding-image-builds.html)
* [Resolução de problemas S2I](https://docs.openshift.com/container-platform/4.5/support/troubleshooting/troubleshooting-s2i.html)
* [Criando imagens](https://docs.openshift.com/container-platform/4.5/openshift_images/create-images.html)

----
<p align="center"><a href="../aula02">❮ Aula anterior</a> | <a href="../aula04">Próxima aula ❯</a></p>