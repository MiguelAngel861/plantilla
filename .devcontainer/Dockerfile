# Usar la imagen base de Arch Linux
FROM archlinux:latest

# Inicializar claves GPG de pacman y actualizar el sistema
RUN pacman-key --init && \
    pacman-key --populate archlinux && \
    pacman -Syu --noconfirm --needed && \
    pacman -S --noconfirm --needed \
        git \
        base-devel \
        zsh \
        neovim \
        curl \
        wget \
        python \
        python-pip \
        make \
        cmake \
        ninja \
        clang \
        gdb \
        lldb \
        fastfetch \
        eza \
        bat \
        jdk21-openjdk \
        sudo

# Crear un usuario sin privilegios para yay (makepkg no se ejecuta como root)
RUN useradd -m -G wheel -s /bin/zsh damian && \
    echo "damian ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Configurar contraseña para el usuario damian (contraseña: 'pass')
RUN echo "damian:pass" | chpasswd

# Cambiar al usuario damian
USER damian
WORKDIR /home/damian
# Instalar Oh My Zsh (modo no interactivo)
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

# Instalar plugins populares de Zsh
RUN git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions && \
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting && \
    git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions

# Configurar Zsh con tema por defecto y plugins
RUN sed -i 's/^ZSH_THEME=.*/ZSH_THEME="lambda"/' ~/.zshrc && \
    sed -i 's/^plugins=.*/plugins=(git zsh-autosuggestions zsh-syntax-highlighting zsh-completions sudo copypath copyfile)/' ~/.zshrc && \
    echo "autoload -U compinit && compinit" >> ~/.zshrc && \
    echo "ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=60'" >> ~/.zshrc

# Instalar yay desde el AUR
RUN git clone https://aur.archlinux.org/yay.git && \
    cd yay && \
    makepkg -si --noconfirm && \
    cd .. && \
    rm -rf yay

# Instalar lazygit usando yay
RUN yay -S --noconfirm lazygit

# Instalar fuentes base necesarias
USER root
RUN pacman -S --noconfirm --needed ttf-dejavu ttf-liberation ttf-roboto-mono powerline-fonts fontconfig

# Actualizar cache de fuentes
RUN fc-cache -f -v

# Limpiar cache de yay como usuario damian
USER damian
RUN yay -Scc --noconfirm

# Volver a root para limpieza del sistema
USER root

# Limpiar cache de pacman y archivos temporales
RUN pacman -Scc --noconfirm && \
    rm -rf /var/cache/pacman/pkg/* /tmp/*

# Establecer Zsh como shell por defecto
RUN chsh -s $(which zsh) damian

# Configuración de Java
ENV LANG=C.UTF-8
ENV JAVA_HOME=/usr/lib/jvm/java-21-openjdk
ENV PATH="${JAVA_HOME}/bin:${PATH}"

# Variable de entorno para terminal básica
ENV TERM="xterm-256color"

# Volver al usuario damian
USER damian
WORKDIR /home/damian

# Establecer Zsh como shell predeterminado al iniciar el contenedor
CMD [ "zsh" ]
