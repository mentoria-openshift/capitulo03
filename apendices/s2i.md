# Apêndice: Instalação da ferramenta do S2I

## Pelo GitHub (todos os sistemas operacionais)
É possível baixar diretamente fo Github, e com Linux e Mac é possível usar os gerenciadores de pacotes (`yum`, `apt` ou `brew`, por exemplo), mas para unificar o processo de instalação, vamos usar a versão binária gerada do código no repositório do GitHub.

### Manualmente
1. Acesse https://github.com/openshift/source-to-image/releases
2. Baixe a versão de acordo com o seu sistema (Darwin para sistemas MacOS, e Linux varia de arquitetura)
3. Extraia o conteúdo do arquivo numa pasta
4. Coloque a pasta onde os binários de encontram na variável PATH
5. O S2I está pronto para ser usado

### Pelo shell (Linux e Mac)
```bash
# Baixe os arquivos (verifique o link correto para sua arquitetura)
wget https://github.com/openshift/source-to-image/releases/download/v1.3.1/source-to-image-v1.3.1-a5a77147-linux-amd64.tar.gz

# Extraia o conteúdo
tar xvzf source-to-image-v1.3.1-a5a77147-linux-amd64.tar.gz 

# Mova os arquivos para uma pasta
sudo mv s2i sti /opt/openshift/bin

# Inclua esta linha no seu ~/.bashrc
export PATH="$PATH:/opt/openshift/bin"
```

