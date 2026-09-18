# Debian 13 Stable + XFCE + X11 — exemplo de instalação

> Objetivo: testar e instalar **Debian 13 Stable + XFCE + X11**, confirmar que o hardware funciona e, só depois, substituir o Ubuntu 26 no NVMe.
>
> Cenário: PC mais antigo, NVIDIA GTX 1050, Windows em outro SSD, uso de Docker/Laravel/MySQL/Remmina/MPV e ferramentas de desenvolvimento.

---

## 1. Antes de começar

Faça backup de tudo que importa no Ubuntu:

- projetos;
- `~/.ssh`;
- `~/.gitconfig`;
- arquivos pessoais;
- bancos/volumes Docker importantes;
- configurações que queira preservar.

Confira seus discos antes de instalar:

```bash
lsblk -o NAME,SIZE,MODEL,FSTYPE,MOUNTPOINTS

```

Anote qual é:

- o **NVMe com Ubuntu**;
- o **SSD com Windows**;
- o **pendrive do Debian**.

**Não apague nenhuma partição enquanto estiver apenas testando.**

---

# PARTE A — Testar pelo Live USB

## 2. Inicialize pelo pendrive

Reinicie o PC e abra o Boot Menu da placa-mãe.

Em placas ASUS normalmente:

```text
F8 = Boot Menu
Del/F2 = BIOS

```

Escolha a entrada do pendrive que começa com:

```text
UEFI:

```

No menu do Debian escolha o modo **Live**.

Não clique em instalar ainda.

---

## 3. Teste básico do hardware

Use o Debian Live por pelo menos 30–60 minutos.

Teste:

- resolução correta do monitor;
- mouse e teclado;
- internet Ethernet/Wi-Fi;
- áudio;
- Bluetooth;
- pendrive/USB;
- navegador e reprodução de vídeo;
- abrir/mover/maximizar janelas repetidamente;
- deixar o PC parado alguns minutos;
- reiniciar pelo menu normalmente.

### Ver hardware detectado

```bash
lspci -k

```

Para USB:

```bash
lsusb

```

Kernel:

```bash
uname -a

```

Erros importantes do boot atual:

```bash
sudo journalctl -b -p err

```

Erros do kernel:

```bash
sudo dmesg --level=err,warn

```

Alguns warnings são normais. O que interessa principalmente é:

- kernel panic;
- GPU reset/error repetitivo;
- I/O error;
- NVMe error;
- filesystem error;
- travamentos completos.

Se o Debian Live ficar estável enquanto o Ubuntu 26 costuma congelar, é um ótimo sinal.

---

# PARTE B — Instalação no NVMe

## 4. Proteja o Windows

A opção mais segura é:

1. desligar completamente o PC;
2. desconectar temporariamente o SSD que contém o Windows, se isso for fácil no seu gabinete;
3. deixar conectado apenas:
   - NVMe que atualmente contém Ubuntu;
   - pendrive do Debian.

Isso impede que o instalador coloque arquivos de boot na partição EFI do Windows.

Se não quiser desconectar o SSD, confira **com muita atenção** o modelo/tamanho de cada disco no instalador.

---

## 5. Instale o Debian

Inicie novamente pelo Live USB e abra:

```text
Install Debian

```

No instalador:

### Idioma

Pode usar:

```text
Português (Brasil)

```

ou inglês, se preferir mensagens técnicas em inglês.

### Desktop

Use:

```text
XFCE

```

### Disco

Escolha **somente o NVMe onde hoje está o Ubuntu 26**.

Se não precisa preservar Ubuntu:

```text
Apagar disco / usar disco inteiro

```

Confirme pelo **modelo e capacidade**, não apenas por `/dev/nvme0n1`.

**Não selecione o SSD do Windows.**

### Filesystem

Para simplicidade:

```text
ext4

```

Não precisa de Btrfs, ZFS ou particionamento complexo para esse objetivo.

### Bootloader

Instale no NVMe do Debian.

Finalize a instalação e reinicie sem o pendrive.

---

# PARTE C — Primeira inicialização

## 6. Atualize tudo

```bash
sudo apt update
sudo apt full-upgrade -y
sudo reboot

```

Depois confirme:

```bash
cat /etc/debian_version
uname -r

```

---

## 7. Confirme os repositórios de firmware

Debian 13 já inclui firmware não livre nas imagens oficiais.

Veja seus repositórios:

```bash
cat /etc/apt/sources.list

```

As entradas principais devem conter, além de `main`:

```text
contrib non-free non-free-firmware

```

Exemplo:

```text
deb http://deb.debian.org/debian trixie main contrib non-free non-free-firmware
deb http://security.debian.org/debian-security trixie-security main contrib non-free non-free-firmware
deb http://deb.debian.org/debian trixie-updates main contrib non-free non-free-firmware

```

Depois:

```bash
sudo apt update

```

---

# PARTE D — Drivers e hardware

## 8. CPU / microcode

Descubra a CPU:

```bash
lscpu | grep "Vendor ID"

```

### AMD

```bash
sudo apt install amd64-microcode

```

### Intel

```bash
sudo apt install intel-microcode

```

Depois:

```bash
sudo reboot

```

---

## 9. NVIDIA GTX 1050

Primeiro veja o que está sendo usado:

```bash
lspci -k | grep -A 3 -E "VGA|3D"

```

No Live USB provavelmente será utilizado `nouveau`.

Após a instalação, instale a ferramenta de detecção:

```bash
sudo apt install nvidia-detect
nvidia-detect

```

A GTX 1050 é da geração **Pascal**.

### Importante

Não baixe o instalador `.run` manualmente do site da NVIDIA.

Para começar, use somente pacotes gerenciados pelo APT.

O Debian 13 possui `nvidia-driver` nos repositórios e ele suporta Pascal:

```bash
sudo apt install nvidia-driver firmware-misc-nonfree
sudo reboot

```

Depois confira:

```bash
nvidia-smi

```

e:

```bash
lspci -k | grep -A 3 -E "VGA|3D"

```

Você deve ver algo semelhante a:

```text
Kernel driver in use: nvidia

```

### Observação importante em 2026

O driver NVIDIA 550 empacotado no Debian 13 suporta a GTX 1050, mas essa série deixou de receber manutenção upstream.

Por isso, para o **primeiro teste de estabilidade**, não complique a instalação com repositórios externos.

1. teste o Debian Live;
2. instale Debian;
3. teste o driver Debian;
4. se tudo estiver estável, decidimos depois se vale migrar para uma branch NVIDIA mais nova que ainda suporte Pascal.

Não misture drivers de Debian 12, Debian Testing ou instaladores `.run`.

---

## 10. Rede

Veja interfaces:

```bash
ip link

```

Veja os dispositivos e drivers:

```bash
lspci -k | grep -A 3 -Ei "network|ethernet"

```

Teste internet:

```bash
ping -c 4 debian.org

```

Se funciona, não mexa em driver.

---

## 11. Áudio

Liste saídas:

```bash
wpctl status

```

Teste também pela interface:

```text
Configurações → Áudio

```

Reproduza algum vídeo/música.

---

## 12. Bluetooth

Confira:

```bash
systemctl status bluetooth

```

Se necessário:

```bash
sudo systemctl enable --now bluetooth

```

Interface gráfica:

```text
Configurações → Bluetooth

```

Teste seu QCY T13 e reinicie o computador pelo menos uma vez com ele pareado.

---

## 13. NVMe / SSD

Liste:

```bash
lsblk -o NAME,SIZE,MODEL,FSTYPE,MOUNTPOINTS

```

Veja erros NVMe:

```bash
sudo dmesg | grep -iE "nvme|I/O error|filesystem error"

```

Se quiser consultar SMART:

```bash
sudo apt install smartmontools
sudo smartctl --scan

```

Depois use o dispositivo retornado, por exemplo:

```bash
sudo smartctl -a /dev/nvme0

```

---

# PARTE E — Programas de trabalho

## 14. Pacotes básicos

```bash
sudo apt install \
  git \
  curl \
  wget \
  unzip \
  zip \
  build-essential \
  ca-certificates \
  gnupg \
  remmina \
  remmina-plugin-rdp \
  mpv

```

### Terminal Kitty

Instale pelo repositório do Debian:

```bash
sudo apt install kitty
kitty
```

Depois de confirmar que ele abre, vá em **Configurações → Aplicativos padrão → Utilitários → Emulador de terminal** e selecione **Kitty**. Se não aparecer na lista, escolha a opção personalizada e informe `kitty` como comando.

Teste se o XFCE passou a abrir o Kitty:

```bash
exo-open --launch TerminalEmulator
```

Em **Configurações → Teclado → Atalhos de aplicativos**, confira o atalho `Super+Enter`: se ele chamar `xfce4-terminal` diretamente, troque o comando por `exo-open --launch TerminalEmulator`.

Para que programas que usam o comando genérico do Debian também abram o Kitty, execute `sudo update-alternatives --config x-terminal-emulator` e selecione a entrada do Kitty, caso esteja disponível.

---

## 15. Remmina

Abra:

```bash
remmina

```

Crie uma conexão:

```text
Protocol: RDP
Server: IP/nome da máquina Windows
Username: usuário do Windows

```

Se conecta normalmente, está pronto.

---

## 16. MPV

Teste:

```bash
mpv video.mkv

```

Para confirmar aceleração/decodificação:

```bash
mpv --hwdec=auto video.mkv

```

---

# PARTE F — Docker / Laravel

## 17. Docker Engine

Prefira **Docker Engine**, não Docker Desktop.

Use o repositório oficial da Docker para Debian.

Remova pacotes conflitantes caso existam:

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do
  sudo apt-get remove "$pkg" 2>/dev/null || true
done

```

Adicione a chave:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg \
  | sudo tee /etc/apt/keyrings/docker.asc > /dev/null
sudo chmod a+r /etc/apt/keyrings/docker.asc

```

Adicione o repositório:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```

Instale:

```bash
sudo apt update

sudo apt install \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin

```

Teste:

```bash
sudo docker run hello-world

```

Permita Docker sem `sudo`:

```bash
sudo usermod -aG docker "$USER"

```

Faça logout/login ou reinicie.

Depois:

```bash
docker version
docker compose version

```

---

## 18. Laravel

Se seus projetos usam Sail/Docker, você não precisa instalar versões específicas de PHP/MySQL diretamente no Debian.

Dentro de um projeto:

```bash
docker compose up -d

```

ou:

```bash
./vendor/bin/sail up -d

```

Teste:

```bash
docker ps

```

Se PHP, MySQL etc. estão nos containers, o host Debian permanece simples e estável.

---

# PARTE G — Verificação final

## 19. Checklist

Antes de considerar a migração concluída:

### Sistema

- 5+ boots sem erro;
- nenhum freeze;
- desligar/reiniciar funciona;
- XFCE funciona suavemente;
- resolução correta.

### Hardware

- NVIDIA funcionando;
- áudio funcionando;
- Ethernet/Wi-Fi;
- Bluetooth;
- teclado/mouse;
- USB;
- NVMe sem erros;
- SSD do Windows intacto.

### Trabalho

- Git;
- SSH;
- Docker Engine;
- Docker Compose;
- projeto Laravel;
- MySQL/container;
- Cursor/editor;
- agents/CLI;
- Remmina/RDP.

### Multimídia

- MPV;
- `.mkv`;
- áudio;
- vídeo em tela cheia.

---

# PARTE H — Se ocorrer freeze/crash

Depois de reiniciar, veja erros do boot anterior:

```bash
sudo journalctl -b -1 -p warning

```

Somente erros:

```bash
sudo journalctl -b -1 -p err

```

Kernel anterior:

```bash
sudo journalctl -k -b -1

```

Procure especificamente por:

```bash
sudo journalctl -b -1 | grep -iE \
"panic|segfault|nvidia|nouveau|nvme|I/O error|amdgpu|oom|watchdog|mce"

```

Se Debian 13 ficar semanas funcionando e os crashes desaparecerem, isso será uma evidência muito forte de que o problema estava no stack/software do Ubuntu 26.

Se o mesmo tipo de crash continuar no Debian, aí investigue hardware: RAM, NVMe/SSD, GPU, fonte ou BIOS.

---

# PARTE I — Confirmar a combinação Debian + XFCE + X11

Antes de considerar a instalação concluída, rode:

```bash
printf "Desktop: %s\n" "$XDG_CURRENT_DESKTOP"
printf "Sessão: %s\n" "$XDG_SESSION_TYPE"
uname -r
```

Esperado:

```text
Desktop: XFCE
Sessão: x11
```

Depois confira o driver gráfico:

```bash
lspci -k | grep -A 3 -E "VGA|3D"
```

Com o driver proprietário NVIDIA instalado, procure:

```text
Kernel driver in use: nvidia
```

Se esses itens estiverem corretos, sua pilha principal será:

```text
Debian 13 Stable
↓
XFCE
↓
X11 / Xorg
↓
driver NVIDIA
↓
GTX 1050
```

Essa é a baseline recomendada para testar estabilidade antes de adicionar customizações ou componentes mais novos.

---

# Configuração recomendada

Para este PC:

```text
Debian 13 Stable
XFCE
X11 / Xorg
ext4
kernel padrão do Debian
driver NVIDIA via APT
Docker Engine
Pacotes de firmware oficiais do Debian

```

Evite inicialmente:

```text
Testing/Sid
kernel de backports
PPAs/repos aleatórios
drivers NVIDIA .run
Wayland
custom kernels

```

A ideia é manter a máquina **boring e previsível**. Primeiro obtenha estabilidade; depois customize.

Depois de confirmar a instalação, siga [RESTAURACAO.md](RESTAURACAO.md) para importar os dados e as preferências do backup.
