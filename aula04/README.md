<p align="center"><a href="../aula03">❮ Aula anterior</a> | <a href="https://github.com/mentoria-openshift/capitulo04">Próximo capítulo ❯</a></p>
<br/>

# Aula 4 - Criando uma imagem de compilação S2I
Nesta aula, usaremos os conceitos aprendidos nas aulas anteriores para criar uma imagem existente do Source-to-Image. 

## A ferramenta S2I
O processo de compilação do S2I necessita de uma imagem com as depêndencias básicas da linguagem em questão para que possa funcionar. Por vezes, as imagens padrão do OpenShift podem não suportar a linguagem ou plataforma específica que o desenvolvedor precisa, o que cria a necessidade de uma imagem própria. Mas para que esta imagem funcione corretamente, ela precisa ser testada. O OpenShift fornece um comando específico para criar e testar imagens S2I. 

O comando `create` da ferramenta S2I serve para criar imagens. A própria ferramenta já gera todo o esqueleto da imagem, com Dockerfile, pastas necessárias, e esboços de scripts S2I e até mesmo um esboço do Dockerfile. Para criar uma imagem, usa-se o comando

```bash
s2i create nome_da_imagem pasta_destino
```

A pasta informada no comando será criada, já com os arquivos base dentro. O desenvolvedor precisa apenas adaptá-los para que suas necessidades sejam supridas.

```
pasta_destino/
├── Dockerfile
├── Makefile
├── README.md
├── s2i
│   └── bin
│       ├── assemble
│       ├── run
│       ├── save-artifacts
│       └── usage
└── test
    ├── run
    └── test-app
        └── index.html
```

Ao término da adaptação dos arquivos gerados, será apenas necessário compilar a imagem, da mesma forma que qualquer outra.

```
podman build -t minha_imagem .
```

Com a imagem de compilação pronta e compilada, é hora de testá-la. Para isso, criaremos uma aplicação de teste na máquina local, que usará a imagem de compilação para ser implantada. O comando `build` da ferramenta S2I serve para gerar uma imagem da aplicação baseada na imagem de compilação, usando um processo semalhante ao que o OpenShift faz internamente ao usar esta estratégia.

```
s2i build pasta_com_codigo_fonte imagem_gerada nome_da_imagem_de_saida
```

Este comando gerará uma imagem contendo sua aplicação já compilada, emulando o processo que o próprio OCP faz. Após isso, basta executar a sua imagem gerada como qualquer outra. Note que o parâmetro identificado como "pasta_com_codigo_fonte" pode ser. além de um diretório local do seu computador, um repositório git.

```
podman run --name minha_aplicacao nome_da_imagem_de_saida
```

Sua aplicação subirá localmente, e você será capaz de testá-la e verificar se sua imagem S2I funcionou corretamente. Para subir sua imagem de compilação S2I para o OpenShift, use os mesmos passos da aula 1 deste capítulo: suba-a para um registro e importe-a para o OCP. Em seguida, use a notação apresentada na aula 3 para criar a aplicação usando sua imagem.

## Exercícios
Isso conclui a aula de criação de imagens S2I no OpenShift. Para praticar o conteúdo aprendido, vamos fazer um exercício prático.

* [Exercício prático](exercicio-pratico.md)

## Referências
* [Documentação do OpenShift](https://docs.openshift.com/)
* [Estratégias de compilação](https://docs.openshift.com/container-platform/4.5/builds/build-strategies.html)
* [Entendendo compilação de imagens](https://docs.openshift.com/container-platform/4.5/builds/understanding-image-builds.html)
* [Resolução de problemas S2I](https://docs.openshift.com/container-platform/4.5/support/troubleshooting/troubleshooting-s2i.html)
* [Criando imagens](https://docs.openshift.com/container-platform/4.5/openshift_images/create-images.html)

----
<p align="center"><a href="../aula02">❮ Aula anterior</a> | <a href="https://github.com/mentoria-openshift/capitulo04">Próximo capítulo ❯</a></p>