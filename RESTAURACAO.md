# Restauração depois de instalar o novo sistema

Instale o sistema e confirme que disco, rede e sessão gráfica funcionam antes de importar dados. Este roteiro usa o backup feito no Ubuntu para restaurar no Debian 13 com XFCE. Em outra distribuição, adapte os pacotes e os caminhos dos aplicativos.

Temas, dotfiles e outras personalizações do rice não fazem parte da restauração básica. Se quiser mantê-los, use um repositório separado e aplique o rice somente depois de instalar e validar o sistema básico.

## Localize o backup

Mantenha o backup em um disco que **não foi formatado**. Ajuste o caminho abaixo para o local onde ele está montado; `~/backup-ubuntu` também serve se você já o copiou para o novo sistema.

```bash
BACKUP="/media/$USER/Backup/backup-ubuntu"
test -d "$BACKUP/obsidian/vaults" && ls "$BACKUP"
```

Não continue se o teste falhar. Antes de mesclar arquivos, confira o destino e faça uma cópia das configurações novas que você já alterou. Nos comandos com `rsync --ignore-existing`, arquivos novos são copiados e arquivos que já existem no destino são preservados; compare esses conflitos manualmente.

### Quando o backup estiver em um arquivo `.tar.zst`

Nesta migração, o backup foi transferido como `~/Downloads/backup-ubuntu.tar.zst`. Não é necessário despejar o arquivo inteiro sobre o novo `/home`: liste o conteúdo e extraia somente o aplicativo ou dado que será restaurado.

```bash
BACKUP_ARCHIVE="$HOME/Downloads/backup-ubuntu.tar.zst"
tar --zstd -tf "$BACKUP_ARCHIVE" | sed -n '1,40p'
```

Os caminhos dentro do arquivo começam com `backup-ubuntu/`. Para usar os comandos desta página, extraia o arquivo inteiro para uma pasta temporária, ou adapte os comandos com `tar --zstd -xOf`/`--strip-components` conforme os exemplos da seção de Kitty e shell abaixo.

## O que existe no backup atual

Inventário de `~/backup-ubuntu` em 18/09/2026. A pasta pode mudar depois de outro backup.

| Pasta | Conteúdo encontrado | Uso na restauração |
| --- | --- | --- |
| `projetos/projects/` | Projetos de desenvolvimento | Copiar para `~/projects/` e validar cada projeto. |
| `pessoal/Documents/`, `pessoal/Pictures/` | Documentos, diário, imagens e outra cópia do Obsidian | Recuperar dados pessoais; usar `obsidian/vaults/` como fonte do vault. |
| `obsidian/vaults/`, `obsidian/config-obsidian/` | Vault `Personal`, inclusive `.obsidian/`, e perfil global | Importar o vault; consultar o perfil global se necessário. |
| `cursor/config-Cursor/`, `cursor/.cursor/` | Preferências, atalhos, extensões, skills, plugins e estado do Cursor | Restaurar preferências e extensões após instalar o aplicativo. |
| `remmina/` | Configuração e perfis de conexão | Restaurar depois de instalar o Remmina. |
| `ptyxis-para-kitty/` | Configuração e sessões do Kitty; exportação do Ptyxis | Importar o Kitty e ajustar os caminhos das sessões. |
| `shell/`, `git/`, `ssh/` | Dotfiles, Starship, Git e chaves/configuração SSH | Comparar e importar por arquivo; manter as permissões SSH. |
| `atalhos/`, `configs/dconf-ubuntu.txt` | Atalhos e preferências do GNOME antigo | Referência para recriar no XFCE; não carregar o dump inteiro. |
| `configs/` | Cópias amplas de `~/.config`, `~/.local/share` e `~/.local/state`; cron, hosts e outros dados | Recuperar apenas o aplicativo ou dado necessário. |
| `listas/` | Pacotes APT, Flatpak e Snap, extensões GNOME, hardware e serviços | Consultar; reinstalar apenas o que faz sentido no novo sistema. |
| `docker/`, `fonts/` | **Vazias** nesta cópia | Não há dumps/volumes nem fontes nessas pastas. `configs/local-share-completo/fonts/` contém fontes de usuário. |

O backup atual também **não contém** as listas opcionais de extensões do VS Code, pacotes npm/Composer ou diagnóstico do Flutter citadas em [BACKUP.md](BACKUP.md). Confira essas ausências antes de formatar o sistema antigo.

## 1. Dados pessoais e projetos

Copie primeiro os arquivos que não dependem de um aplicativo instalado. `--ignore-existing` preserva arquivos que você já criou no sistema novo.

```bash
mkdir -p "$HOME/projects" "$HOME/Documents" "$HOME/Pictures"
rsync -a --ignore-existing "$BACKUP/projetos/projects/" "$HOME/projects/"
rsync -a --ignore-existing --exclude='Obsidian/' "$BACKUP/pessoal/Documents/" "$HOME/Documents/"
rsync -a --ignore-existing "$BACKUP/pessoal/Pictures/" "$HOME/Pictures/"
```

Confira os projetos, os arquivos `.env` e as permissões antes de executar qualquer aplicação. O backup completo do `/home`, se existir em outro disco, serve para resgatar arquivos esquecidos; não o despeje inteiro sobre o novo `/home`.

## 2. Obsidian

Com o Obsidian fechado, copie o vault. A pasta `.obsidian/` dentro dele guarda preferências, aparência, atalhos e plugins do vault. Depois abra a pasta `~/Documents/Obsidian/Personal` como vault no Obsidian.

```bash
mkdir -p "$HOME/Documents/Obsidian"
rsync -a --ignore-existing "$BACKUP/obsidian/vaults/" "$HOME/Documents/Obsidian/"
```

`obsidian/config-obsidian/` é um perfil global antigo, com cache e caminhos da instalação anterior. Guarde-o para consulta; importe algum item dele apenas se faltar algo após abrir o vault. A cópia em `pessoal/Documents/Obsidian/` é redundante.

## 3. Terminal, shell, Git e SSH

Instale o Kitty e o Zsh conforme [o guia de instalação](INSTALACAO-DEBIAN.md#terminal-kitty). Com o aplicativo fechado, copie sua configuração e sessões:

```bash
mkdir -p "$HOME/.config/kitty"
rsync -a --ignore-existing "$BACKUP/ptyxis-para-kitty/kitty/" "$HOME/.config/kitty/"
```

Se o backup ainda estiver compactado, a restauração seletiva usada nesta migração é:

```bash
mkdir -p "$HOME/.config"
tar --zstd -x -f "$BACKUP_ARCHIVE" -C "$HOME/.config" \
  --strip-components=2 'backup-ubuntu/ptyxis-para-kitty/kitty'
```

Isso restaura `kitty.conf` e as dez sessões (`local-*`, `remoto-*` e `padrao.kitty-session`). O `kitty.conf` inicia no perfil `arthur` e oferece `F7`, depois `P`, para escolher uma sessão. As sessões remotas dependem das chaves e do `~/.ssh/config` já restaurados. Personalizações visuais devem ser aplicadas depois, a partir do repositório separado de dotfiles.

Leia `ptyxis-para-kitty/README.md` **no backup**: as sessões têm caminhos `~/projects/...` e conexões SSH que podem precisar de ajuste. A fonte usada é JetBrainsMono Nerd Font. Restaure as fontes de usuário encontradas no backup:

```bash
mkdir -p "$HOME/.local/share/fonts"
rsync -a --ignore-existing "$BACKUP/configs/local-share-completo/fonts/" "$HOME/.local/share/fonts/"
fc-cache -f
```

Com o arquivo compactado, use:

```bash
mkdir -p "$HOME/.local/share/fonts"
tar --zstd -x -f "$BACKUP_ARCHIVE" -C "$HOME/.local/share/fonts" \
  --strip-components=3 'backup-ubuntu/configs/local-share-completo/fonts'
fc-cache -f
```

No XFCE, configure `Super+Enter` para abrir o Kitty. O atalho usado nesta migração foi:

```bash
xfconf-query -c xfce4-keyboard-shortcuts \
  -p '/commands/custom/<Super>Return' -n -t string -s kitty
```

Em `shell/dotfiles/` estão `.bashrc`, `.zshrc` e `.profile`; em `shell/starship.toml` está a configuração do prompt. Compare cada arquivo com o novo sistema antes de copiar. Por exemplo, use `diff -u "$HOME/.zshrc" "$BACKUP/shell/dotfiles/.zshrc"` e só então copie os trechos desejados. Faça o mesmo com `git/.gitconfig`; o backup também tem `configs/config-completo/starship.toml`.

Para restaurar somente o `.zshrc` diretamente do arquivo compactado:

```bash
tar --zstd -x -f "$BACKUP_ARCHIVE" -C "$HOME" \
  --strip-components=3 'backup-ubuntu/shell/dotfiles/.zshrc'
zsh -n "$HOME/.zshrc"
```

O `.zshrc` antigo referencia Starship, Cursor, NVM, `zsh-autosuggestions`, `zsh-syntax-highlighting` e `~/bin`. Se esses componentes ainda não existirem, comente as linhas e identifique-as como provenientes do backup; reative-as somente depois de instalar as dependências. Para habilitar sugestões e destaque no Debian, consulte [Sugestões e autocomplete no Zsh](INSTALACAO-DEBIAN.md#sugestoes-e-autocomplete-no-zsh). O arquivo restaurado nesta migração foi tratado dessa forma.

As chaves estão em `ssh/.ssh/`. Copie sem substituir arquivos SSH já criados no sistema novo:

```bash
install -d -m 700 "$HOME/.ssh"
rsync -a --ignore-existing "$BACKUP/ssh/.ssh/" "$HOME/.ssh/"
chmod 700 "$HOME/.ssh"
chmod 600 "$HOME/.ssh/id_ed25519"
```

Se um arquivo já existia, compare as versões e decida qual usar. Teste a conexão com o host correspondente; verifique também `config`, `known_hosts` e a chave pública antes de usá-los.

## 4. Cursor, extensões e Remmina

Instale e abra o Cursor uma vez, depois feche-o. Faça uma cópia de `~/.config/Cursor/User` e `~/.cursor` se você já configurou algo no sistema novo. Importe as preferências e extensões salvas:

```bash
mkdir -p "$HOME/.config/Cursor/User" "$HOME/.cursor/extensions"
rsync -a "$BACKUP/cursor/config-Cursor/User/" "$HOME/.config/Cursor/User/"
rsync -a "$BACKUP/cursor/.cursor/extensions/" "$HOME/.cursor/extensions/"
```

O backup também inclui skills, plugins e agentes do Cursor. Se ainda forem compatíveis com a versão instalada, copie-os com o aplicativo fechado:

```bash
rsync -a --ignore-existing "$BACKUP/cursor/.cursor/skills/" "$HOME/.cursor/skills/"
rsync -a --ignore-existing "$BACKUP/cursor/.cursor/skills-cursor/" "$HOME/.cursor/skills-cursor/"
rsync -a --ignore-existing "$BACKUP/cursor/.cursor/plugins/" "$HOME/.cursor/plugins/"
rsync -a --ignore-existing "$BACKUP/cursor/.cursor/agents/" "$HOME/.cursor/agents/"
```

`cursor/.cursor/cli-config.json`, `argv.json`, `chats/`, `projects/` e o restante de `cursor/config-Cursor/` preservam configuração ou estado da instalação antiga; consulte-os apenas se precisar recuperar algo específico. Reabra o Cursor e confira settings, atalhos, skills e extensões.

Instale o Remmina e feche-o antes de copiar os perfis:

```bash
mkdir -p "$HOME/.config/remmina" "$HOME/.local/share/remmina"
rsync -a "$BACKUP/remmina/config-remmina/" "$HOME/.config/remmina/"
rsync -a "$BACKUP/remmina/local-share-remmina/" "$HOME/.local/share/remmina/"
```

Quando o backup estiver no arquivo `~/Downloads/backup-ubuntu.tar.zst`, feche o Remmina, preserve a configuração atual e extraia somente as pastas do aplicativo:

```bash
BACKUP_ARCHIVE="$HOME/Downloads/backup-ubuntu.tar.zst"
STAMP=$(date +%Y%m%d-%H%M%S)

cp -a "$HOME/.config/remmina" "$HOME/.config/remmina.before-restore-$STAMP" 2>/dev/null
cp -a "$HOME/.local/share/remmina" "$HOME/.local/share/remmina.before-restore-$STAMP" 2>/dev/null
mkdir -p "$HOME/.config/remmina" "$HOME/.local/share/remmina"

tar --zstd -x -f "$BACKUP_ARCHIVE" \
  -C "$HOME/.config/remmina" \
  --strip-components=3 'backup-ubuntu/remmina/config-remmina'

tar --zstd -x -f "$BACKUP_ARCHIVE" \
  -C "$HOME/.local/share/remmina" \
  --strip-components=3 'backup-ubuntu/remmina/local-share-remmina'
```

Abra o Remmina e confira os perfis e as conexões restaurados. Senhas podem depender do chaveiro do sistema anterior e precisar ser informadas novamente.

## 5. Outros aplicativos e preferências

`configs/config-completo/`, `configs/local-share-completo/` e `configs/local-state-completo/` guardam cópias amplas da instalação antiga. Verifique essas pastas quando faltar uma configuração. Há, por exemplo, `configs/config-completo/google-chrome/` (perfis e extensões do Chrome), `composer/`, `evolution/`, `configs/local-share-completo/flatpak/` e `keyrings/`. Reinstale o aplicativo, abra-o uma vez, feche-o e recupere somente a pasta relevante. Perfis de navegador e chaveiros podem conter credenciais; trate-os como dados privados e confira a compatibilidade antes de importar.

Use `listas/apt-manual-packages.txt`, `flatpak-apps.txt` e `snap-packages.txt` para escolher quais aplicativos reinstalar. Não execute essas listas como um script: nomes e fontes de pacotes mudam entre distribuições. `listas/gnome-extensions.txt`, `configs/local-share-completo/gnome-shell/extensions/`, `atalhos/` e `configs/dconf-ubuntu.txt` registram extensões e preferências do GNOME antigo. No XFCE, use-os como referência para recriar atalhos. Se voltar para GNOME, revise cada extensão antes de instalá-la e importe somente os atalhos que ainda fazem sentido.

Em `configs/` também há `crontab.txt`, `cron.d/` e `hosts`. Revise as entradas e restaure somente as que ainda são necessárias. As listas de Docker descrevem containers, imagens e volumes antigos, mas a pasta `docker/` vazia não permite recuperar dados de volumes ou bancos; busque dumps e cópias reais em outro backup antes de depender deles.

## Conferência final

- [ ] Documentos, imagens, projetos e vault do Obsidian abrem no novo sistema.
- [ ] Kitty, shell, Git, SSH, Cursor e Remmina funcionam com as configurações escolhidas.
- [ ] Extensões necessárias do Cursor e plugins do vault, se houver, aparecem e funcionam.
- [ ] Bancos, volumes Docker e fontes têm uma fonte de recuperação confirmada, se forem necessários.
- [ ] O backup original continua guardado fora do disco formatado.
