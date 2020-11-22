<p align="center"><a href="../aula01">Aula 1</a></p>
<br/>

# Exercício prático - Aula 1

Para esta aula, usaremos nossa querida, estimada e valorosa aplicação `simplecrud`. Sim, a mesma dos exercícios anteriores. É necessário também que você tenha acesso a um cluster do OpenShift na versão 4.5 Você pode usar a Katacoda para criar cluster que duram uma hora, usar os labs no site da Red Hat ou até mesmo seguir nosso [apêndice sobre como instalar o CRC](apendices/openshift_crc.md). Caso você possa investir num cluster para aprendizado, a licença do OpenShift é gratuita por 60 dias e você pode hospedá-lo na Azura, AWS, IBM Cloud ou na Red Hat, mas a infraestruturas e as máquinas onde o OpenShift será hospedado não são gratuitas, e os recursos necessários são vastos. Para mais detalhes, veja a [seção de teste do OpenShift](https://www.openshift.com/try).

## Requerimentos
- Você deve ter acesso de desenvolvedor a um cluster do OpenShift
- Será necessário ter uma conta no Quay.io
- Seu projeto deverá se chamar capitulo03
- Sua aplicação deverá se chamar aula01
- Sua aplicação deverá ser criada com uma configuração de implantação
- A imagem da aplicação deverá estar hospedada no Quay.io
- A imagem da aplicação deverá ser chamada simplecrud, com a tag aula01
- Seu Dockerfile deverá clonar o código contido em https://github.com/mentoria-openshift/simplecrud-spring.git

## Passo-a-passo
Este passo a passo indica como chegar ao resultado final do exercício. Clique na seta ao lado do enunciado para exibir a resposta. Note que o exercício todo será feito na linha de comando.

<details> 
  <summary>1. Use como base o Dockerfile construído no capítulo 1, na aula 2</summary>
   
```Dockerfile
```

</details>

<details> 
  <summary>2. Faça as correções necessárias de permissões para o OpenShift</summary>
  
```Dockerfile
```

</details>

<details> 
  <summary>3. Compile a imagem e copie-a para o Quay.io</summary>

```bash
```

</details>

<details> 
  <summary>4. Faça login no OpenShift e crie o projeto</summary>
  
```bash
```

</details>

<details> 
  <summary>5. Crie a aplicação usando como base o endereço da imagem</summary>
   
```bash
```

</details>

<details> 
  <summary>6. Exponha o serviço da aplicação numa rota externa</summary>
   
```bash
```

</details>

<details> 
  <summary>7. Teste a aplicação</summary>

```bash
```

</details>

---
<p align="center"><a href="../aula01">Aula 1</a></p>