# Backup antes de trocar de distribuição

> Objetivo: preservar configurações, conexões, projetos, fontes, ferramentas e dados importantes antes de apagar o sistema atual. O exemplo deste guia usa Ubuntu → Debian 13.
>
> Estratégia recomendada: **backup seletivo + backup completo do `/home` como segurança**.

## Caminho rápido

Antes de apagar o sistema antigo, sempre:

1. escolha um destino fora do disco que será formatado;
2. copie dados únicos, projetos e credenciais que não possam ser recriados;
3. mantenha uma cópia completa do `/home` como segurança;
4. abra arquivos do backup e confira se eles estão legíveis.

As seções de aplicativos e ferramentas são condicionais: execute somente as que correspondem ao que você usa.

## Índice

- Preparação e dados essenciais
  - [1. Pasta principal](#1-criar-uma-pasta-principal-de-backup)
  - [2. Projetos](#2-projetos-de-desenvolvimento)
  - [3. SSH](#3-ssh)
  - [4. Git](#4-git)
- Aplicativos e preferências
  - [5. Cursor](#5-cursor)
  - [6. Remmina](#6-remmina)
  - [7. Obsidian](#7-obsidian)
  - [8. Fontes](#8-fontes)
  - [9. Shell e terminal](#9-shell--terminal)
  - [14. Flatpak](#14-flatpak)
  - [15. Snap](#15-snap)
  - [16. Configurações de aplicativos](#16-configurações-de-aplicativos)
  - [17. GNOME e atalhos](#17-gnome--atalhos-atuais-do-ubuntu)
  - [18. VS Code](#18-vs-code)
  - [24. Navegador](#24-navegador)
- Desenvolvimento
  - [10. Docker](#10-docker)
  - [11. Bancos MySQL](#11-bancos-mysql)
  - [12. Volumes Docker](#12-docker-volumes-importantes)
  - [19. Node e npm](#19-node--npm)
  - [20. Composer](#20-composer)
  - [21. Flutter](#21-flutter)
  - [22. Chaves Android e Flutter](#22-android--flutter-signing-keys)
  - [23. Credenciais e secrets](#23-credenciais-e-secrets)
- Sistema e diagnóstico
  - [13. Pacotes APT](#13-pacotes-apt-instalados)
  - [25. Serviços habilitados](#25-lista-de-serviços-habilitados)
  - [26. Cron jobs](#26-cron-jobs)
  - [27. Hosts personalizados](#27-hosts-personalizados)
  - [28. Hardware](#28-lista-geral-do-hardware)
- Conferência e restauração
  - [29. Backup completo do HOME](#29-backup-completo-do-home)
  - [30. Verificar o backup](#30-verificar-o-backup)
  - [Checklist final](#checklist-final-antes-de-apagar-o-ubuntu)
  - [Regra de restauração](#regra-para-restaurar-no-debian)

---

## 1. Criar uma pasta principal de backup

Escolha um disco externo, outro SSD ou pendrive grande.

Exemplo:

```bash
mkdir -p ~/backup-ubuntu
```

Estrutura mínima criada pelos exemplos deste guia:

```text
backup-ubuntu/
├── projetos/
├── ssh/
├── git/
├── cursor/
├── remmina/
├── obsidian/
├── fonts/
├── shell/
├── docker/
├── configs/
└── listas/
```

O backup usado na migração atual possui subdivisões adicionais, como `projetos/projects/`, `obsidian/vaults/`, `cursor/config-Cursor/` e `pessoal/`. Consulte o [inventário do backup restaurado](RESTAURACAO.md#o-que-existe-no-backup-atual) antes de adaptar os comandos de restauração. Em outro backup, mantenha uma estrutura consistente e ajuste os caminhos dos dois guias em conjunto.

---

## 2. Projetos de desenvolvimento

Copie seus projetos completos.

Inclua:

- projetos Laravel;
- projetos Flutter;
- projetos React;
- scripts;
- arquivos `.env`;
- certificados locais;
- chaves/API keys armazenadas localmente;
- arquivos fora do Git.

Exemplo:

```bash
cp -a ~/Projetos ~/backup-ubuntu/projetos/
```

Ajuste o caminho conforme sua estrutura real.

---

## 3. SSH

Muito importante.

Copie:

```bash
cp -a ~/.ssh ~/backup-ubuntu/ssh/
```

Isso preserva:

- chaves privadas;
- chaves públicas;
- `known_hosts`;
- `config`;
- hosts SSH personalizados.

Depois da migração, mantenha permissões corretas:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_*
```

Não aplique `chmod 600` em arquivos `.pub`.

---

## 4. Git

Copie:

```bash
cp ~/.gitconfig ~/backup-ubuntu/git/
```

Se existir:

```bash
cp -a ~/.config/git ~/backup-ubuntu/git/
```

---

## 5. Cursor

Antes de copiar:

**Feche completamente o Cursor.**

Salve:

```bash
cp -a ~/.config/Cursor ~/backup-ubuntu/cursor/
```

Também:

```bash
cp -a ~/.cursor ~/backup-ubuntu/cursor/
```

Se existir:

```bash
cp -a ~/.cursor/extensions ~/backup-ubuntu/cursor/
```

Isso pode preservar:

- settings;
- keybindings;
- snippets;
- extensões;
- estado local;
- configurações do Agent CLI;
- chats/estado interno dependendo da versão.

Arquivos particularmente importantes:

```text
~/.config/Cursor/User/settings.json
~/.config/Cursor/User/keybindings.json
~/.config/Cursor/User/snippets/
~/.cursor/
```

---

## 6. Remmina

Feche o Remmina antes do backup.

Copie:

```bash
cp -a ~/.config/remmina ~/backup-ubuntu/remmina/ 2>/dev/null
```

E:

```bash
cp -a ~/.local/share/remmina ~/backup-ubuntu/remmina/ 2>/dev/null
```

Isso pode preservar:

- conexões RDP;
- servidores;
- usuários;
- preferências;
- perfis.

---

## 7. Obsidian

### Vaults

Copie todos os seus vaults completos.

Muito importante preservar também a pasta:

```text
.obsidian/
```

dentro de cada vault.

Ela contém:

- plugins;
- temas;
- configurações;
- hotkeys;
- workspace;
- aparência.

Exemplo:

```bash
cp -a ~/Documentos/Obsidian ~/backup-ubuntu/obsidian/
```

Ajuste o caminho.

### Configuração global

Também copie:

```bash
cp -a ~/.config/obsidian ~/backup-ubuntu/obsidian/ 2>/dev/null
```

---

## 8. Fontes

Fontes instaladas pelo usuário:

```bash
cp -a ~/.local/share/fonts ~/backup-ubuntu/fonts/ 2>/dev/null
```

Se existir:

```bash
cp -a ~/.fonts ~/backup-ubuntu/fonts/ 2>/dev/null
```

Também gere uma lista:

```bash
fc-list > ~/backup-ubuntu/listas/fontes-instaladas.txt
```

---

## 9. Shell / Terminal

Salve:

```bash
cp ~/.bashrc ~/backup-ubuntu/shell/ 2>/dev/null
cp ~/.profile ~/backup-ubuntu/shell/ 2>/dev/null
cp ~/.bash_aliases ~/backup-ubuntu/shell/ 2>/dev/null
cp ~/.zshrc ~/backup-ubuntu/shell/ 2>/dev/null
```

Se usa Oh My Zsh:

```bash
cp -a ~/.oh-my-zsh ~/backup-ubuntu/shell/ 2>/dev/null
```

Se usa Starship:

```bash
cp ~/.config/starship.toml ~/backup-ubuntu/shell/ 2>/dev/null
```

---

## 10. Docker

### Ver containers

```bash
docker ps -a
```

### Ver volumes

```bash
docker volume ls
```

### Ver imagens

```bash
docker images
```

Salve essas listas:

```bash
docker ps -a > ~/backup-ubuntu/listas/docker-containers.txt
docker volume ls > ~/backup-ubuntu/listas/docker-volumes.txt
docker images > ~/backup-ubuntu/listas/docker-images.txt
```

---

## 11. Bancos MySQL

Se os bancos importantes estão em Docker, prefira gerar dumps SQL.

Exemplo:

```bash
docker exec NOME_CONTAINER \
  mysqldump -u root -p --all-databases \
  > ~/backup-ubuntu/docker/mysql-all-databases.sql
```

Se usa MySQL instalado diretamente no Ubuntu:

```bash
mysqldump -u root -p --all-databases \
  > ~/backup-ubuntu/docker/mysql-all-databases.sql
```

Teste se o arquivo foi criado:

```bash
ls -lh ~/backup-ubuntu/docker/mysql-all-databases.sql
```

Não confie somente em copiar containers.

---

## 12. Docker volumes importantes

Se algum projeto guarda dados importantes somente em volumes Docker, faça backup deles individualmente.

Primeiro:

```bash
docker volume ls
```

Exemplo de backup:

```bash
docker run --rm \
  -v NOME_DO_VOLUME:/volume \
  -v ~/backup-ubuntu/docker:/backup \
  alpine \
  tar czf /backup/NOME_DO_VOLUME.tar.gz -C /volume .
```

---

## 13. Pacotes APT instalados

Não copie os pacotes propriamente ditos.

Gere uma lista:

```bash
apt-mark showmanual \
  > ~/backup-ubuntu/listas/apt-manual-packages.txt
```

Também:

```bash
dpkg --get-selections \
  > ~/backup-ubuntu/listas/dpkg-selections.txt
```

Essas listas servem como referência.

**Não reinstale cegamente todos os pacotes no Debian.**

Ubuntu e Debian possuem diferenças de nomes e versões.

---

## 14. Flatpak

Lista:

```bash
flatpak list --app --columns=application \
  > ~/backup-ubuntu/listas/flatpak-apps.txt
```

Se usa configurações importantes de aplicativos Flatpak:

```bash
cp -a ~/.var/app ~/backup-ubuntu/configs/flatpak-app-data
```

Restaurar isso deve ser feito seletivamente.

---

## 15. Snap

Se usa Snap:

```bash
snap list > ~/backup-ubuntu/listas/snap-packages.txt
```

No Debian, prefira reinstalar os aplicativos via:

- APT;
- Flatpak;
- AppImage;
- pacote oficial.

Não é necessário levar o Snap para o Debian.

---

## 16. Configurações de aplicativos

Configurações de usuário geralmente ficam em:

```text
~/.config/
~/.local/share/
~/.local/state/
```

Não recomendo restaurar essas pastas inteiras no Debian.

Em vez disso, faça uma cópia de segurança completa:

```bash
cp -a ~/.config ~/backup-ubuntu/configs/config-completo
```

```bash
cp -a ~/.local/share ~/backup-ubuntu/configs/local-share-completo
```

```bash
cp -a ~/.local/state ~/backup-ubuntu/configs/local-state-completo 2>/dev/null
```

Depois restaure **somente aplicativos específicos** conforme necessário.

---

## 17. GNOME / atalhos atuais do Ubuntu

Como você vai para XFCE, não restaure as configurações do GNOME por inteiro.

Mas salve uma referência:

```bash
dconf dump / \
  > ~/backup-ubuntu/configs/dconf-ubuntu.txt
```

Extensões GNOME:

```bash
gnome-extensions list \
  > ~/backup-ubuntu/listas/gnome-extensions.txt
```

Isso serve apenas para consulta futura.

O backup usado neste roteiro também contém `atalhos/` com os atalhos exportados do GNOME e `ptyxis-para-kitty/` com a configuração e as sessões convertidas para o Kitty. Ao copiar o backup para outro disco, mantenha essas duas pastas junto das demais; os READMEs dentro delas explicam o que foi exportado.

---

## 18. VS Code

Se também usa VS Code:

```bash
cp -a ~/.config/Code ~/backup-ubuntu/configs/ 2>/dev/null
```

Extensões:

```bash
code --list-extensions \
  > ~/backup-ubuntu/listas/vscode-extensions.txt
```

---

## 19. Node / npm

Liste pacotes globais:

```bash
npm list -g --depth=0 \
  > ~/backup-ubuntu/listas/npm-global.txt
```

Se usa NVM:

```bash
cp -a ~/.nvm ~/backup-ubuntu/configs/ 2>/dev/null
```

Em geral é melhor reinstalar NVM/Node limpos no Debian.

---

## 20. Composer

Liste pacotes globais:

```bash
composer global show \
  > ~/backup-ubuntu/listas/composer-global.txt 2>/dev/null
```

Configuração:

```bash
cp -a ~/.config/composer ~/backup-ubuntu/configs/ 2>/dev/null
```

---

## 21. Flutter

Confira instalação:

```bash
flutter doctor -v \
  > ~/backup-ubuntu/listas/flutter-doctor.txt
```

Se Flutter estiver instalado manualmente em alguma pasta, anote:

```bash
which flutter
```

Melhor prática:

**reinstalar Flutter limpo no Debian** e preservar apenas projetos/configurações realmente importantes.

---

## 22. Android / Flutter signing keys

Se desenvolve Android, verifique:

```text
~/.android/
```

Backup:

```bash
cp -a ~/.android ~/backup-ubuntu/configs/ 2>/dev/null
```

Muito importante verificar também arquivos `.jks` ou `.keystore`.

Procure:

```bash
find ~ -type f \( -name "*.jks" -o -name "*.keystore" \) 2>/dev/null
```

Não perca chaves de assinatura de apps publicados.

---

## 23. Credenciais e secrets

Verifique manualmente:

```text
.env
.env.local
credentials.json
*.pem
*.key
*.crt
*.p12
*.jks
*.keystore
```

Procure:

```bash
find ~ -type f \( \
  -name ".env" \
  -o -name "*.pem" \
  -o -name "*.key" \
  -o -name "*.p12" \
  -o -name "*.jks" \
  -o -name "*.keystore" \
\) 2>/dev/null
```

Revise os resultados antes de formatar.

---

## 24. Navegador

Se usa sincronização do Chrome/Firefox, confirme que está logado e sincronizado.

Se quiser backup local do Chrome:

```bash
cp -a ~/.config/google-chrome ~/backup-ubuntu/configs/ 2>/dev/null
```

Chromium:

```bash
cp -a ~/.config/chromium ~/backup-ubuntu/configs/ 2>/dev/null
```

Firefox:

```bash
cp -a ~/.mozilla ~/backup-ubuntu/configs/ 2>/dev/null
```

Restaurar perfis completos entre distribuições deve ser feito com cuidado.

---

## 25. Lista de serviços habilitados

Útil como referência:

```bash
systemctl list-unit-files --state=enabled \
  > ~/backup-ubuntu/listas/systemd-enabled.txt
```

---

## 26. Cron jobs

Usuário atual:

```bash
crontab -l \
  > ~/backup-ubuntu/configs/crontab.txt 2>/dev/null
```

Sistema:

```bash
sudo cp -a /etc/cron.d ~/backup-ubuntu/configs/cron.d 2>/dev/null
```

---

## 27. Hosts personalizados

Se você editou:

```text
/etc/hosts
```

salve:

```bash
sudo cp /etc/hosts ~/backup-ubuntu/configs/hosts
sudo chown "$USER":"$USER" ~/backup-ubuntu/configs/hosts
```

---

## 28. Lista geral do hardware

Muito útil para comparar Ubuntu vs Debian.

```bash
lspci -k > ~/backup-ubuntu/listas/lspci-k.txt
```

```bash
lsusb > ~/backup-ubuntu/listas/lsusb.txt
```

```bash
lsblk -o NAME,SIZE,MODEL,FSTYPE,MOUNTPOINTS \
  > ~/backup-ubuntu/listas/lsblk.txt
```

```bash
uname -a > ~/backup-ubuntu/listas/uname.txt
```

---

## 29. Backup completo do HOME

Além do backup seletivo, recomendo fortemente ter uma cópia completa do seu `/home`.

Exemplo para um disco externo montado em:

```text
/media/SEU_USUARIO/Backup
```

Use:

```bash
rsync -aAXHv --info=progress2 \
  "$HOME/" \
  "/media/$USER/Backup/home-$USER/"
```

Ajuste o caminho do destino.

Isso preserva inclusive arquivos escondidos.

### Importante

Não copie esse `/home` inteiro de volta por cima do Debian.

Use-o como:

> seguro caso você descubra depois que esqueceu algum arquivo.

---

## 30. Verificar o backup

Antes de formatar:

```bash
du -sh ~/backup-ubuntu
```

Confira manualmente:

```bash
ls -lah ~/backup-ubuntu
```

Se estiver em disco externo:

```bash
ls -lah /media/$USER/NOME_DO_DISCO
```

Abra alguns arquivos aleatórios para confirmar que são legíveis.

---

## Checklist final antes de apagar o Ubuntu

### Desenvolvimento

- [ ] projetos;
- [ ] `.env`;
- [ ] SSH;
- [ ] Git;
- [ ] certificados;
- [ ] Android signing keys;
- [ ] scripts.

### Cursor

- [ ] `~/.config/Cursor`;
- [ ] `~/.cursor`;
- [ ] extensions;
- [ ] settings;
- [ ] keybindings;
- [ ] snippets.

### Aplicativos

- [ ] Remmina;
- [ ] Obsidian;
- [ ] fontes;
- [ ] navegador;
- [ ] VS Code, se usado.

### Docker

- [ ] lista de containers;
- [ ] lista de volumes;
- [ ] dumps MySQL;
- [ ] volumes importantes.

### Sistema

- [ ] lista APT;
- [ ] lista Flatpak;
- [ ] lista Snap;
- [ ] dconf;
- [ ] serviços;
- [ ] crontab;
- [ ] `/etc/hosts`;
- [ ] hardware atual.

### Segurança

- [ ] cópia completa do `/home`;
- [ ] backup armazenado fora do NVMe que será formatado;
- [ ] arquivos importantes abrem normalmente;
- [ ] SSD do Windows não será formatado.

---

## Regra para restaurar no Debian

Evite:

```text
copiar todo ~/.config de volta
copiar todo ~/.local de volta
reinstalar todos os pacotes Ubuntu automaticamente
```

Prefira:

```text
instalar aplicativo limpo
↓
abrir uma vez
↓
fechar
↓
restaurar somente a configuração daquele aplicativo
↓
testar
```

Isso reduz muito a chance de carregar configurações incompatíveis ou problemas do Ubuntu 26 para o Debian.

---

[← Índice geral](README.md) · [Próxima etapa: instalar o Debian →](INSTALACAO-DEBIAN.md)
