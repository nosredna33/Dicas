
# Rerar um Repositório RPM de uma URL de um repositório

Este script é útil quando você tem uma rede lenta ou isolada por segurança. Você vai cpm isso criar um repositório baseado em arquivos no S.O., ao invés de baixa da rede. Util em ambientes DEV.

### extraction.sh

Responsável de gerar a lista com os `.rpm`s

```bash
#!/bin/bash
# ------------------------------------------------------------------
# Script: create-local-centos-vault.sh
# Objetivo: Baixar RPMs do Vault CentOS 7.9, criar repo local e ZIP
# Requisitos: wget, java + Tika, createrepo, zip
# ------------------------------------------------------------------

# Diretório de trabalho temporário
WORKDIR="/cygdrive/d/TEMP/RH7-RPMs"
mkdir -p "$WORKDIR"
cd "$WORKDIR" || exit 1

# URL base do Vault
VAULT_URL="https://vault.centos.org/7.9.2009/os/x86_64/Packages/"

# Baixa a página HTML do diretório Packages
wget -O packages.html "$VAULT_URL"

# Caminho do Apache Tika (modifique conforme seu ambiente)


wget -O packages.html https://vault.centos.org/7.9.2009/os/x86_64/Packages/

#!/bin/bash
# ------------------------------------------------------------------
# Script: create-local-centos-vault.sh
# Objetivo: Baixar RPMs do Vault CentOS 7.9, criar repo local e ZIP
# Requisitos: wget, java + Tika, createrepo, zip
# ------------------------------------------------------------------


# Caminho do Apache Tika (modifique conforme seu ambiente)
export TIKA_BIN=/cygdrive/d/Projetos/MegaPower/bin/tika.jar

# Diretório de trabalho temporário
WORKDIR="$HOME/RH7-RPMs"
mkdir -p "$WORKDIR"
cd "$WORKDIR" || exit 1

# URL base do Vault
VAULT_URL="https://vault.centos.org/7.9.2009/os/x86_64/Packages/"

# Baixa a página HTML do diretório Packages
wget -O packages.html "$VAULT_URL"


# Extrai nomes dos RPMs e cria lista completa de URLs
java -jar ${TIKA_BIN} --text ./packages.html | \
   grep ".rpm" | \
   sed -E 's/^(.*)\.rpm\s+.*$/\1.rpm/g' | \
   awk '{printf("%s\n", $0);}' | \
   sed -E 's/\s+//g' | \
   awk -v base="$VAULT_URL" '{print base $0}' \
   > lista.txt

# Cria diretório para download
mkdir -p rpms
cd rpms || exit 1

# Baixa todos os RPMs da lista (continua se interrompido)
wget -c -i ../lista.txt
 
# Volta para diretório raiz do trabalho
cd "$WORKDIR" || exit 1

# Compacta todos os RPMs em um ZIP
zip -9 -r Packages-x86_64.zip rpms/*
``` 
Agora leve o pacote para a máquina de destino e execute lá o próximo script

### criaLocalRepo.sh

```bash
# Cria repositório local para yum
LOCAL_REPO_DIR="/var/localrepo"
mkdir -p "$LOCAL_REPO_DIR"
cd "$LOCAL_REPO_DIR/"

unzip Packages-x86_64.zip

# Gera metadata do repo
createrepo "$LOCAL_REPO_DIR"

# Cria arquivo .repo para yum
cat > ./CentOS-Local.repo << EOF
[localrepo]
name=CentOS 7 Local Repo
baseurl=file://$LOCAL_REPO_DIR/
enabled=1
gpgcheck=0
EOF

echo "==========================================="
echo "Repositório local criado com sucesso!"
echo "RPMs em: $LOCAL_REPO_DIR"
echo "Arquivo repo: $WORKDIR/CentOS-Local.repo"
echo "ZIP backup: $WORKDIR/Packages-x86_64.zip"
echo "Use 'yum --disablerepo=\* --enablerepo=localrepo install <pacote>' para instalar pacotes."
echo "==========================================="
```

