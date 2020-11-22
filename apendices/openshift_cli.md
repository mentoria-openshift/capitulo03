# Apêndice: Instalação do OpenShift CLI

## Windows
1. Acesse o [arquivo de downloads](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/) do OC e baixe a versão desejada (o curso se baseia na verão 4.5.7)
2. Extraia os arquivos e coloque-os numa pasta (exemplo `C:\Arquivos de programas\OpenShift\bin`)
3. Adicione a pasta onde os arquivos foram colocados ao `PATH`.
    * Abra o menu iniciar e digite "var".
    * Selecione a opção de varáveis de ambiente de usuário
    * Busque a variável `PATH` e clique nela duas vezes
    * Adicione uma entrada com o valor da pasta que você criou (exemplo `C:\Arquivos de programas\OpenShift\bin`)
    * Aplique todas as mudanças e abra uma sessão do terminal para utilizar o OC.

## Linux e MacOS
1. Acesse o [arquivo de downloads](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/) do OC e baixe a versão desejada (o curso se baseia na verão 4.5.7)
2. Extraia os arquivos e coloque-os numa pasta (exemplo `/opt/openshift/bin`)
3. Adicione a pasta onde os arquivos foram colocados ao `PATH`.
    ```bash
    # Inclua esta linha no seu ~/.bashrc
    export PATH="$PATH:/opt/openshift/bin"
    ```

4. Abra uma nova sessão do terminal ou use `source ~/.bashrc` para ativar as mudanças na mesma sessão usada.