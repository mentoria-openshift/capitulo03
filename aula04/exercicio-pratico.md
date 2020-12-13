<p align="center"><a href="../aula03">Aula 3</a> | <a href="../aula04">Aula 4</a>
<br/>

# Exercício prático - Aulas 3 e 4

Este exercício colocará em prática as coisas ensinadas na aula 3. Aqui você modificará uma imagem S2I e fará a implantação dela a partir de seu código fonte. Ao término desta aula, você será capaz de entender os conceitos do S2I e como modificar uma imagem. Na próxima aula, criaremos uma imagem do zero a partir da ferramenta do S2I.

## Requerimentos
- Você deve ter acesso de desenvolvedor a um cluster do OpenShift
- Será necessário ter uma conta no GitHub
- Seu projeto deverá se chamar capitulo03
- Sua aplicação deverá se chamar aula04
- Sua aplicação deverá ser criada com uma configuração de implantação
- O código da aplicação deverá estar hospedada no GitHub (forkado do simplecrud do curso para sua conta)
- O objeto JSON para teste da aplicação estará disponível do endpoint `/api`
- Use o fluxo de imagem S2I `openjdk-11-rhel8:1.0` para compilação da aplicação.
- Os scripts originais não devem ser executados, e sim sobrepostos pelos customizados.
- Você deverá commitar suas alterações de scripts na branch `s2i-scripts`.
- Execute 2 passos de compilação: `mvn clean install -DskipTests` para compilar e `mvn test` para testes unitários.

## Passo-a-passo
Este passo a passo indica como chegar ao resultado final do exercício. Clique na seta ao lado do enunciado para exibir a resposta. Note que o exercício todo será feito na linha de comando.

<details> 
  <summary>1. Crie a nova branch e a pasta dos scripts</summary>
  
```bash
# Esses comandos devem ser executados na pasta raiz da aplicação
git checkout -b s2i-scripts
mkdir -p .s2i/bin
```

</details>

<details> 
  <summary>2. Crie os scripts <code>assemble</code>, <code>save-artifacts</code> e <code>run</code> nas pastas recém criadas</summary>
  
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

Save artifacts:
```bash
#!/bin/bash

if [ -d ${HOME}/.m2 ]; then
    pushd ${HOME}
    echo "--> Salvando artefatos de compilação..."
    tar cf - ${HOME}/.m2
    popd
fi
```

</details>

<details> 
  <summary>3. Suba as alterações, crie o projeto e a aplicação</summary>
  
```bash
# Commit das alterações
git add .
git commit -m "criação de scripts s2i"
git push -u origin s2i-scripts

# Criação dos recursos
oc new-project capitulo03
oc new-app --name aula04 openjdk-11-rhel8:1.0~https://github.com/seu_usuario/simplecrud-spring.git#s2i-scripts

# Acompnhamento de compilação
oc logs -f bc/aula04
watch -n 1 oc get pod
```

</details>

<details> 
  <summary>4. Exponha o serviço após a implantação e teste a aplicação</summary>
  
```bash
oc expose svc aula04
oc get route

curl aula04-capitulo03.apps.cluster.com/api
{"hostname":"simplecrud-1-f7zn2","Mensagem a todos":"Mas para onde vai o mundo?","Banco de dados":"H2","Perfil da aplicação":"default","OpenShift é muito":"maneiro","Mensagem":"Olá, Mundo!","Está aplicação está rodando em":"português"}
```

</details>

---
<p align="center"><a href="../aula03">Aula 3</a> | <a href="../aula04">Aula 4</a>