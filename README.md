# Guia Definitivo para Automação Pós-Instalação do Debian

## Introdução: Automatizando Sua Estação de Trabalho Debian

O objetivo deste relatório é fornecer um script de automação robusto e detalhado, projetado para transformar uma instalação nova do sistema operacional Debian em um ambiente de trabalho completo e rico em funcionalidades para desenvolvedores e usuários avançados. A automação deste processo de configuração inicial oferece benefícios significativos, incluindo consistência entre múltiplas instalações, eficiência ao economizar tempo e esforço manual, e a aderência a boas práticas de segurança e manutenção do sistema.

Este documento está estruturado em três partes principais. A primeira seção aborda a configuração fundamental do sistema, preparando as bases para as instalações subsequentes. A segunda parte foca na instalação de aplicações de desktop e ferramentas de mídia essenciais. A terceira seção detalha a configuração de um ambiente de desenvolvimento moderno. Finalmente, o relatório culmina na apresentação de um script unificado que integra todas essas etapas, seguido por instruções cruciais para a finalização da configuração.

## Seção 1: Construindo a Base: Configuração do Sistema e do APT

Esta seção aborda os passos iniciais e críticos que preparam o sistema para a instalação de software subsequente. Estas ações são pré-requisitos para quase todas as outras tarefas descritas no script.

### 1.1 Habilitando Acesso Completo a Software: Ativando os Repositórios `contrib`, `non-free` e `non-free-firmware`

O Debian, em sua essência, possui um forte compromisso filosófico com o software livre, o que se reflete diretamente na estrutura de seus repositórios de pacotes. Por padrão, apenas o repositório `main`, que contém software 100% livre, é ativado. No entanto, para uma estação de trabalho moderna, é frequentemente necessário acessar softwares que não se enquadram nesses critérios estritos. Os repositórios `contrib`, `non-free` e `non-free-firmware` contêm pacotes que são essenciais para a funcionalidade completa do sistema, como drivers de hardware proprietários, firmware de dispositivos e certos codecs de multimídia.

A ativação desses repositórios não é apenas uma questão de preferência, mas uma dependência fundamental para a instalação de pacotes solicitados posteriormente, como o utilitário `unrar` e o pacote auxiliar `libdvd-pkg`, que residem nos componentes `non-free` e `contrib`, respectivamente. O script de automação abordará isso de forma programática, utilizando um editor de fluxo como o `sed` para modificar o arquivo `/etc/apt/sources.list` de maneira não destrutiva, adicionando os componentes `contrib non-free non-free-firmware` a cada linha de fonte de pacotes relevante. Após a modificação, o comando `sudo apt update` será executado para atualizar o cache de pacotes e tornar o novo software disponível para instalação.

### 1.2 Estabelecendo Privilégios Seguros: Instalando e Configurando o `sudo`

Diferentemente de distribuições como o Ubuntu, o Debian não instala o pacote `sudo` por padrão se uma senha para o usuário `root` for definida durante a instalação do sistema. O uso do `sudo` para tarefas administrativas rotineiras é uma prática de segurança recomendada, pois oferece vantagens significativas sobre o login direto como `root`. Entre os benefícios estão o registro detalhado de quais comandos foram executados por qual usuário, a capacidade de conceder permissões granulares e a redução do risco de danos acidentais ao sistema.

O script de automação verificará primeiro se o `sudo` já está instalado. Caso não esteja, ele utilizará o comando `su -c "..."` para executar `apt install -y sudo` com privilégios de `root`, solicitando a senha de `root` se necessário. O passo central é adicionar o usuário atual ao grupo `sudo`, o que será feito com o comando `adduser $USER sudo`. Este é o método padrão e recomendado para conceder privilégios administrativos completos em um sistema Debian.

É fundamental compreender que as alterações na filiação de grupo de um usuário só entram em vigor em uma nova sessão de login. Portanto, após a execução do script, será necessário que o usuário faça logout e login novamente, ou reinicie o sistema, para que as permissões do `sudo` sejam aplicadas corretamente. Embora o script não vá modificar diretamente o arquivo `/etc/sudoers`, é importante notar que qualquer edição manual desse arquivo deve ser feita exclusivamente com o comando `visudo`, que realiza uma verificação de sintaxe para prevenir erros que poderiam bloquear o acesso administrativo ao sistema.

## Seção 2: Capacidades Essenciais de Desktop e Mídia

Esta seção foca em equipar o ambiente de trabalho com aplicações centrais para navegação na web, gerenciamento de arquivos e consumo de mídia, transformando o sistema base em uma máquina funcional para o uso diário.

### 2.1 Instalando o Google Chrome Através do Repositório Oficial

O Google Chrome é um software proprietário e, por essa razão, não está incluído nos repositórios oficiais do Debian. Existem dois métodos principais para sua instalação: baixar o pacote `.deb` manualmente ou adicionar o repositório APT oficial do Google. A abordagem via repositório é tecnicamente superior e o padrão profissional por duas razões principais: garante a autenticidade dos pacotes através da verificação da chave GPG do Google e, mais importante, permite que o navegador seja atualizado automaticamente junto com o resto do sistema através do comando padrão `sudo apt upgrade`. Isso elimina a necessidade de baixar e instalar manualmente cada nova versão.

O script implementará este método superior seguindo os passos oficiais:

1.  Instalar os pacotes de pré-requisito `wget` e `gpg`.
2.  Baixar e adicionar a chave de assinatura do Google ao chaveiro do sistema com o comando:
    ```bash
    wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo gpg --dearmor -o /usr/share/keyrings/google-chrome-keyring.gpg
    ```
3.  Adicionar o repositório do Chrome às fontes do APT, criando um novo arquivo em `/etc/apt/sources.list.d/`, uma prática mais limpa do que editar o arquivo principal:
    ```bash
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome-keyring.gpg] http://dl.google.com/linux/chrome/deb/ stable main" | sudo tee /etc/apt/sources.list.d/google-chrome.list
    ```
4.  Atualizar a lista de pacotes e, finalmente, instalar o navegador:
    ```bash
    sudo apt update && sudo apt install -y google-chrome-stable
    ```

### 2.2 Dominando Arquivos Compactados: Um Conjunto Completo de Utilitários de Compressão

É importante distinguir entre arquivadores, como o `tar`, que agrupam múltiplos arquivos em um só, e compressores, como `gzip`, `bzip2` e `xz`, que reduzem o tamanho dos arquivos. Embora as ferramentas padrão do Linux sejam onipresentes, uma estação de trabalho moderna deve ser capaz de lidar com formatos comuns em outros sistemas operacionais, notadamente `.zip`, `.rar` e `.7z`.

Para garantir compatibilidade máxima, o script instalará um conjunto abrangente de utilitários. Para o formato `.rar`, o pacote `unrar` é a escolha recomendada. Ele é proprietário e reside no repositório `non-free`, fornecendo a capacidade de extrair arquivos `.rar`, o que reforça a importância da configuração realizada na Seção 1.1. O script executará um único comando para instalar todas as ferramentas necessárias: `sudo apt install -y zip unzip unrar 7zip`.

A tabela a seguir resume os pacotes essenciais:

| Formato | Propósito | Pacote(s) Debian | Componente do Repositório |
| :--- | :--- | :--- | :--- |
| **.zip** | Criar e extrair arquivos ZIP | `zip`, `unzip` | `main` |
| **.rar** | Extrair arquivos RAR (proprietário) | `unrar` | `non-free` |
| **.7z** | Criar e extrair arquivos 7-Zip | `7zip` | `main` |
| **.tar.gz, .tar.bz2, .tar.xz** | Arquivos padrão do Linux | `tar`, `gzip`, `bzip2`, `xz` | `main` (pré-instalado) |

### 2.3 Desbloqueando Multimídia: Instalação do VLC e Codecs

O VLC é amplamente reconhecido como um reprodutor de mídia extremamente versátil. A instalação base é direta: `sudo apt install -y vlc`.

No entanto, para uma funcionalidade completa, especialmente a reprodução de DVDs comerciais criptografados, pacotes adicionais são necessários. O Debian oferece o pacote `libdvd-pkg` no repositório `contrib`. Este pacote atua como um auxiliar que baixa o código-fonte da biblioteca `libdvdcss2` e o compila localmente. O script automatizará este processo de duas etapas: primeiro com `sudo apt install -y libdvd-pkg` e, em seguida, executando `sudo dpkg-reconfigure libdvd-pkg`. Adicionalmente, pacotes de codecs como `gstreamer1.0-plugins-bad`, `gstreamer1.0-plugins-ugly` e `lame` serão incluídos para garantir a mais ampla compatibilidade.

## Seção 3: Criando o Kit de Ferramentas do Desenvolvedor

Esta seção é dedicada à configuração de um ambiente de desenvolvimento de software moderno.

### 3.1 O Editor Moderno: Instalando o Visual Studio Code

O Visual Studio Code (VS Code) se estabeleceu como um editor de código líder. Assim como o Chrome, sua instalação é mais bem gerenciada através de um repositório APT oficial.

O processo de instalação espelha o do Google Chrome:

1.  Instalar os pré-requisitos `wget` e `gpg`.
2.  Baixar e adicionar a chave GPG da Microsoft:
    ```bash
    wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
    sudo install -D -o root -g root -m 644 microsoft.gpg /usr/share/keyrings/microsoft-archive-keyring.gpg
    ```
3.  Adicionar o repositório do VS Code:
    ```bash
    sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/usr/share/keyrings/microsoft-archive-keyring.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
    ```
4.  Atualizar e instalar o `code`:
    ```bash
    sudo apt update && sudo apt install -y apt-transport-https code
    ```

Como alternativa, o **VSCodium** é uma compilação de código aberto do VS Code sem a telemetria da Microsoft. O script instalará o pacote oficial `code`.

### 3.2 O Ambiente de Execução JavaScript: Uma Análise Profunda da Instalação do Node.js

Existem três estratégias principais para instalar o Node.js:

  - **Método 1: Repositório Padrão do Debian (`apt install nodejs`):** O mais simples, mas a versão é frequentemente antiga.

  - **Método 2: NodeSource PPA (Abordagem Recomendada):** Utiliza um repositório de terceiros para pacotes atualizados, oferecendo um bom equilíbrio entre modernidade e integração com o sistema. O script usa este método para instalar a versão LTS (Suporte de Longo Prazo).

  - **Método 3: NVM (Node Version Manager):** Um script de shell que gerencia múltiplas versões do Node.js por usuário. Oferece máxima flexibilidade para desenvolvedores que trabalham em múltiplos projetos com diferentes requisitos.

A tabela a seguir compara as metodologias:

| Método | Como Funciona | Controle de Versão | Integração | Ideal Para |
| :--- | :--- | :--- | :--- | :--- |
| **Padrão Debian** | `apt` dos repositórios oficiais. | Versão única e antiga. | Total (nível de sistema). | Testes rápidos. |
| **NodeSource** | Adiciona repositório, depois usa `apt`. | Versão única e atualizada (LTS). | Total (nível de sistema). | Desenvolvimento geral, servidores. |
| **NVM** | Gerencia no diretório pessoal. | Múltiplas versões, alternáveis. | Por usuário, não sistêmico. | Desenvolvedores profissionais. |

## Seção 4: O Script de Automação Unificado

Este script shell consolidado automatiza todo o processo.

```bash

#!/bin/bash
# ==============================================================================
# Script Pós-Instalação Modernizado para Debian
#
# Este script automatiza a instalação de ferramentas essenciais de sistema,
# desktop, multimídia e desenvolvimento com feedback aprimorado.
#
# USO:
# Execute com privilégios de sudo: sudo ./install
# ==============================================================================

# --- Configuração de Robustez ---
# Sai imediatamente se um comando falhar.
set -e
# Trata erros de forma elegante.
trap 'error_handler $? $LINENO' ERR

# --- Funções de Feedback e Log ---
log_step() {
    echo -e "\n\e[1;35m=======================================================================\e[0m"
    echo -e "\e[1;35m>>> PASSO: $1\e[0m"
    echo -e "\e[1;35m=======================================================================\e[0m"
}

log_success() {
    echo -e "\e[1;32m[SUCESSO] $1\e[0m"
}

log_error() {
    echo -e "\e[1;31m[ERRO] $1\e[0m" >&2
}

log_info() {
   echo -e "\e[1;34m[INFO] $1\e[0m"
}

log_warning() {
    echo -e "\e[1;33m[AVISO] $1\e[0m"
}

# --- Tratamento de Erros ---
error_handler() {
    log_error "Ocorreu um erro na linha $2. Código de saída: $1."
    exit 1
}

# --- Funções de Instalação ---
configure_apt_repositories() {
    log_step "Configurando repositórios APT (contrib, non-free, non-free-firmware)"
    log_info "Fazendo backup do /etc/apt/sources.list..."
    cp /etc/apt/sources.list /etc/apt/sources.list.bak
    log_info "Adicionando componentes 'contrib', 'non-free' e 'non-free-firmware'..."

 # Este comando 'sed' substitui de forma inteligente toda a seção de componentes
    # nas linhas 'deb' e 'deb-src', garantindo a sequência correta.
    sed -i -E 's#^(deb(-src)?\s+[^ ]+\s+[^ ]+)\s+.*#\1 main contrib non-free non-free-firmware#' /etc/apt/sources.list

    
    log_info "Atualizando a lista de pacotes..."
    apt update
    log_success "Repositórios APT configurados."
}

install_sudo_and_configure_user() {
    log_step "Verificando e configurando o sudo"
    if ! command -v sudo &> /dev/null; then
        log_info "Comando 'sudo' não encontrado. Instalando..."
        apt install -y sudo
        log_success "'sudo' foi instalado."
    fi

    # Prefere o usuário que chamou sudo, senão usa o logname
    TARGET_USER=${SUDO_USER:-$(logname)}
    if groups "$TARGET_USER" | grep -q '\bsudo\b'; then
        log_info "Usuário '$TARGET_USER' já pertence ao grupo 'sudo'."
    else
        log_info "Adicionando o usuário '$TARGET_USER' ao grupo 'sudo'..."
        adduser "$TARGET_USER" sudo
        log_warning "O usuário '$TARGET_USER' deve fazer logout e login novamente para que as permissões de sudo entrem em vigor."
    fi
    log_success "Configuração do sudo concluída."
}

install_essential_packages() {
    log_step "Instalando pacotes essenciais e multimídia"
    log_info "Instalando: wget, gpg, curl, build-essential, zip, unzip, vlc, python3-pip..."
    apt install -y \
        wget gpg apt-transport-https curl build-essential \
        zip unzip unrar 7zip \
        vlc lame libdvd-pkg \
        gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly \
        python3-pip python3-venv

    log_info "Reconfigurando libdvd-pkg de forma não interativa..."
    dpkg-reconfigure -fnoninteractive libdvd-pkg
    log_success "Pacotes essenciais instalados."
}

install_google_chrome() {
    log_step "Instalando o Google Chrome"
    log_info "Baixando a chave de assinatura do Google..."
    wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | gpg --dearmor -o /usr/share/keyrings/google-chrome-keyring.gpg
    log_info "Adicionando o repositório do Google Chrome..."
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome-keyring.gpg] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list
    log_info "Atualizando a lista de pacotes e instalando o Chrome..."
    apt update
    apt install -y google-chrome-stable
    log_success "Google Chrome instalado."
}

install_vscode() {
    log_step "Instalando o Visual Studio Code"
    log_info "Baixando a chave de assinatura da Microsoft..."
    wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /usr/share/keyrings/microsoft-archive-keyring.gpg
    log_info "Adicionando o repositório do VS Code..."
    echo "deb [arch=amd64,arm64,armhf signed-by=/usr/share/keyrings/microsoft-archive-keyring.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list
    log_info "Atualizando a lista de pacotes e instalando o VS Code..."
    apt update
    apt install -y code
    log_success "Visual Studio Code instalado."
}

install_nodejs_lts() {
    log_step "Instalando o Node.js (versão LTS) via NodeSource"
    log_info "Baixando e executando o script de configuração do NodeSource..."
    curl -fsSL https://deb.nodesource.com/setup_lts.x | bash -
    log_info "Instalando o pacote Node.js..."
    apt-get install -y nodejs
    log_success "Node.js LTS instalado."
}

# --- Função Principal ---
main() {
    if [ "$(id -u)" -ne 0 ]; then
        log_error "Este script precisa ser executado como root ou com privilégios de sudo."
        echo "Exemplo: sudo ./setup_script.sh"
        exit 1
    fi

    configure_apt_repositories
    install_sudo_and_configure_user
    install_essential_packages
    install_google_chrome
    install_vscode
    install_nodejs_lts

    log_step "CONFIGURAÇÃO CONCLUÍDA"
    log_success "O sistema foi configurado com sucesso!"
    log_info "AÇÕES NECESSÁRIAS:"
    log_warning "1. REINICIE o sistema para garantir que todas as alterações, especialmente as permissões do 'sudo', sejam aplicadas corretamente."
    log_info "2. Após reiniciar, verifique as instalações com os comandos:"
    echo "   - google-chrome-stable --version"
    echo "   - code --version"
    echo "   - node -v"
    echo "   - npm -v"
}

# Executa a função principal
main


```

## Seção 5: Finalizando Sua Configuração: Ações Pós-Script e Boas Práticas

Esta seção final fornece instruções de acompanhamento cruciais para garantir que o sistema esteja totalmente operacional.

### 5.1 Aplicando as Alterações do Sistema

Para que a nova filiação do usuário ao grupo `sudo` tenha efeito, um ciclo de **logout e login é obrigatório**. Uma reinicialização completa do sistema é a opção mais segura e recomendada.

### 5.2 Manutenção do Sistema

Execute regularmente `sudo apt update && sudo apt upgrade -y` para manter todo o sistema atualizado. Graças à abordagem de instalação baseada em repositórios, este comando também manterá o Google Chrome, o VS Code e o Node.js atualizados.

Para verificar as versões do software instalado, use:

  - `google-chrome-stable --version`
  - `code --version`
  - `node -v`
  - `npm -v`

### 5.3 Personalização

Este script deve ser visto como um modelo. Sinta-se à vontade para personalizá-lo, comentando funções para omitir programas ou adicionando novas funções para instalar outros softwares.
