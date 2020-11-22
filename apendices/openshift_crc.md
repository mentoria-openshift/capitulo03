# Apêndice: Rodando o OpenShift 4.x com CRC

Antes de prosseguir, note que o OpenShift é pesado, e requer certos recursos mínimos de hardware para que seja executado. Para executar um cluster OpenShift com o CRC, é preciso uma máquina com 4 vCPUs, 8 GB RAM e 35 GB de espaço em disco. No caso de você não ter um computador que suporte o CRC, você pode usar o [portal de aprendizado da Red Hat](https://learn.openshift.com/) para criar um cluster de aprendizado que fica de pé por uma hora. Basta criar uma conta no site e escolher o OpenShift na versão 4.5 para ser compatível com o que faremos.

Caso você possa investir num cluster para aprendizado, a licença do OpenShift é gratuita por 60 dias e você pode hospedá-lo na Azura, AWS, IBM Cloud ou na Red Hat, mas a infraestruturas e as máquinas onde o OpenShift será hospedado não são gratuitas, e os recursos necessários são vastos. Para mais detalhes, veja a [seção de teste do OpenShift](https://www.openshift.com/try).

## Windows
1. Acesse o [arquivo de downloads](https://mirror.openshift.com/pub/openshift-v4/clients/crc/) do CRC e baixe a versão desejada (o curso se baseia na verão 1.15.0).
2. Extraia os arquivos e coloque-os numa pasta (exemplo `C:\Arquivos de programas\OpenShift\bin`).
3. Adicione a pasta onde os arquivos foram colocados ao `PATH`.
    * Abra o menu iniciar e digite "var".
    * Selecione a opção de varáveis de ambiente de usuário.
    * Busque a variável `PATH` e clique nela duas vezes.
    * Adicione uma entrada com o valor da pasta que você criou (exemplo `C:\Arquivos de programas\OpenShift\bin`).
    * Aplique todas as mudanças e abra uma sessão do terminal para utilizar o OC.

## Linux e MacOS
1. Acesse o [arquivo de downloads](https://mirror.openshift.com/pub/openshift-v4/clients/crc/) do CRC e baixe a versão desejada (o curso se baseia na verão 1.15.0)
2. Extraia os arquivos e coloque-os numa pasta (exemplo `/opt/openshift/bin`).
3. Adicione a pasta onde os arquivos foram colocados ao `PATH`.
    ```bash
    # Inclua esta linha no seu ~/.bashrc
    export PATH="$PATH:/opt/openshift/bin"
    ```

4. Abra uma nova sessão do terminal ou use `source ~/.bashrc` para ativar as mudanças na mesma sessão usada.