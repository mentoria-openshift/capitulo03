<p align="center"><a href="../aula01">Aula 1</a>
<br/>

# Questionário - Aulas 1

1. Para que server as labels em imagens do OpenShift?

    **a)** São apenas estética.

    **b)** Identificação da imagem no repositório público.

    **c)** Metadados para o fluxo de imagem e execução da apĺicação.

    **d)** Identificar o criador da imagem.
---
2. Qual o comando para criar uma aplicação a partir de uma imagem?

    **a)** `oc new-app --name nome_da_aplicacao --strategy=docker-image https://registro.com/repositorio/imagem`

    **b)** `oc create app --name nome_da_aplicacao --docker-image registro.com/repositorio/imagem`

    **c)** `oc new-app --name nome_da_aplicacao --docker-image registro.com/repositorio/imagem`

    **d)** `oc create app --name nome_da_aplicacao --docker-image registro.com/repositorio/imagem`
---
3. O que o flag `--as-deployment-config` faz e em qual versão ele foi introduzido?

    **a)** Cria uma aplicação no cluster e foi introduzido na versão 4.5.

    **b)** Cria uma configuração de implantação para a imagem e foi introduzido na versão 3.11.

    **c)** Cria uma aplicação à parte do cluster e foi introduzido na versão 4.2.

    **d)** Cria uma configuração de implantação para a imagem e foi introduzido na versão 4.5.
---
4. É possível, por padrão e sem alterar configurações de acesso, criar aplicações que executem com root?

    **a)** Não, pois o OpenShift requer que um usuário específico não root execute a aplicação.

    **b)** Sim, pois o OpenShift está preparado para executar aplicações com root por padrão.

    **c)** Não, pois o OpenShift requer que um usuário aleatório não root execute a aplicação.

    **d)** Não, pois nenhum usuário deve executar uma aplicação.

    **e)** Sim, mas é necessário que o sudo esteja instalado na imagem.
---

<details> 
  <summary>Respostas</summary>

    1. Resposta: c
    2. Resposta: c
    3. Resposta: d
    4. Resposta: c
</details>

---
<p align="center"><a href="../aula01">Aula 1</a>