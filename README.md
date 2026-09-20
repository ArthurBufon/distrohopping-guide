# Distrohopping guide

![distrohopping-guide](images/def-not-generated-by-ai-lol.png)

Roteiro pessoal para planejar uma troca de distribuição e recuperar dados e preferências. O exemplo atual é **Ubuntu → Debian 13 com XFCE e X11**, em um PC com GTX 1050 e Windows em outro SSD. Na próxima troca, revise os passos específicos da distribuição e do hardware antes de executá-los.

| Etapa | Documento | Quando usar | Resultado esperado |
| --- | --- | --- | --- |
| 1. Backup | [BACKUP.md](BACKUP.md) | Antes de formatar o sistema antigo | Dados importantes copiados e conferidos em outro disco |
| 2. Instalação | [INSTALACAO-DEBIAN.md](INSTALACAO-DEBIAN.md) | Depois de validar o backup | Debian testado, instalado e com hardware e programas básicos verificados |
| 3. Restauração | [RESTAURACAO.md](RESTAURACAO.md) | Depois de confirmar a estabilidade do sistema novo | Dados e configurações necessários importados seletivamente |

Temas, dotfiles e demais personalizações visuais devem ficar em um repositório separado. Faça o rice somente depois de instalar e validar toda a base do sistema; a migração e o rice são etapas independentes.

O [inventário da restauração](RESTAURACAO.md#o-que-existe-no-backup-atual) descreve o que foi encontrado em `~/backup-ubuntu`, inclusive pastas vazias. Este repositório guarda **somente instruções**: projetos, vaults, perfis, chaves, bancos e o backup real ficam fora dele.

## Estado desta migração

Em 19/09/2026, o Debian 13 com XFCE/X11 já estava instalado, com drivers, Git/SSH/GPG e projetos restaurados. O Kitty foi instalado e recebeu os dez perfis convertidos do Ptyxis, as sessões locais/remotas e a fonte JetBrains Mono Nerd Font; o atalho `Super+Enter` do XFCE foi configurado para abrir o Kitty. O Zsh e o `.zshrc` do backup também foram restaurados.

O backup usado nessa etapa foi o arquivo `~/Downloads/backup-ubuntu.tar.zst`. Docker/Compose, Node/npm, Cursor, Obsidian e Starship ainda precisam ser instalados ou restaurados; os plugins de sugestões, autocomplete e destaque de sintaxe do Zsh já foram instalados. O checklist local e temporário em `docs/handoffs/` acompanha esse progresso e não é versionado.
