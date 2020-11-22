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

## CentOS, RHEL e MacOS
1. Acesse o [arquivo de downloads](https://mirror.openshift.com/pub/openshift-v4/clients/crc/) do CRC e baixe a versão desejada (o curso se baseia na verão 1.15.0)
2. Extraia os arquivos e coloque-os numa pasta (exemplo `/opt/openshift/bin`).
3. Adicione a pasta onde os arquivos foram colocados ao `PATH`.
    ```bash
    # Inclua esta linha no seu ~/.bashrc
    export PATH="$PATH:/opt/openshift/bin"
    ```

4. Abra uma nova sessão do terminal ou use `source ~/.bashrc` para ativar as mudanças na mesma sessão usada.

## Debian e Ubuntu
O CRC foi feito para rodar em sistemas Fedora, como o CentOS e o RHEL, que são próprios da Red Hat. Para executá-lo em outras distribuições, configurações adicionais são necessárias. 

1. Acesse o [arquivo de downloads](https://mirror.openshift.com/pub/openshift-v4/clients/crc/) do CRC e baixe a versão desejada (o curso se baseia na verão 1.15.0)
2. Extraia os arquivos e coloque-os numa pasta (exemplo `/opt/openshift/bin`).
3. Adicione a pasta onde os arquivos foram colocados ao `PATH`.
    ```bash
    # Inclua esta linha no seu ~/.bashrc
    export PATH="$PATH:/opt/openshift/bin"
    ```

4. Abra uma nova sessão do terminal ou use `source ~/.bashrc` para ativar as mudanças na mesma sessão usada.
5. Instale as dependências adicionais necessárias
```bash
# Gerenciadores de virtualização e DNS
sudo apt install -y qemu-kvm libvirt-daemon libvirt-daemon-system dnsmasq
```

6. O OpenShift requer o uso do `dnsmasq` ao invés do `systemd-resolved`. Por isso precisamos configurar para que eles rodem em paralelo. 
```bash
# Edite o arquivo do resolvd
sudo vi /etc/systemd/resolved.conf

# Descomente a linha que diz DNS e adicione o IP 127.0.0.2
```

7. Corrija o endereço do `resolved`
```bash
# Certifique-se que a linha que diz listen-address esteja descomentada e com 127.0.0.2 como valor
cat /etc/dnsmasq.conf | grep 127.0.0.2
listen-address=127.0.0.2
```

8. Agora configure o `dnsmasq` para que ele ignore o 'resolved`
```bash
# Edite o arquivo do dnsmasq
sudo vi /etc/default/dnsmasq 

# Certifique que a linha IGNORE_RESOLVCONF está descomentada, e que o valor dela seja yes
IGNORE_RESOLVCONF=yes
```

9. Adicione o CRC à configuração do dnsmasq
```bash
# Edite o arquivo
sudo vi /etc/dnsmasq.d/crc.conf 

# Adicione as informações
server=/crc.testing/192.168.130.1
address=/apps-crc.testing/192.168.130.11

# Reinicie os serviços
sudo systemctl restart systemd-resolved
sudo systemctl restart dnsmasq
```

10. Desative checagens pré-definidas do CRC
```bash
crc config set skip-check-network-manager-installed true
crc config set skip-check-network-manager-config true
crc config set skip-check-network-manager-running true
crc config set skip-check-crc-dnsmasq-file true
```

11. Prossiga para o uso do CRC

## Depois de instalar
1. Acesse a [plataforma do OpenShift](https://cloud.openshift.com) e faça login
2. Clique em **Create Cluster** e selecione OpenShift Container Platform
3. Selecione **Run on Laptop**
4. Copie ou baixe o **Pull Secret**
5. Execute o comando `crc setup` para que seu ambiente seja preparado
6. Digite `crc start` após o comando anterior se completar e cole o pull secret quando pedido, e tecle enter
7. O OpenShift está pronto para ser usado. 
