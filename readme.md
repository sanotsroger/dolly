# Dolly

Script para sincronizar diretórios locais usando [rclone](https://rclone.org/),
com exclusões configuráveis, deleção espelhada opcional (via `--mirror`; por
padrão nada é apagado no destino), múltiplos pares source/destination, modo
dry-run, logs organizados por execução e um resumo estilo `git status`
(adicionado/modificado/removido).

## Instalação

```sh
chmod +x dolly
ln -s "$(pwd)/dolly" ~/.local/bin/dolly   # garanta que ~/.local/bin está no PATH
```

Requer o binário `rclone` instalado e disponível no `PATH`.

## Uso

```sh
dolly init             # cria os arquivos de configuração e diretórios de log
dolly --dry-run        # simula a sincronização (nada é copiado/apagado; atalho: -d)
dolly                  # executa a sincronização de fato (pede confirmação; por padrão, nunca apaga no destino)
dolly -y               # executa sem pedir confirmação
dolly --mirror         # ativa deleção espelhada: apaga no destino o que não existe mais na origem (atalho: -m)
dolly --edit-path      # abre paths.conf no nano (atalho: -ep)
dolly --edit-filter    # abre filters.txt no nano (atalho: -ef)
dolly -h               # ajuda
```

Por padrão cada par roda com `rclone copy` (só copia/atualiza, nunca apaga).
Com `--mirror`/`-m`, roda `rclone sync` (espelha a origem, apagando extras no
destino) — por exemplo: `dolly -y --mirror`.

Qualquer flag não reconhecida pelo dolly (ex.: `--bwlimit 1M`, `--transfers 8`,
`--exclude "*.tmp"`) é repassada direto para o rclone de cada par, por
exemplo: `dolly -y --bwlimit 1M`.

`--edit-path`/`--edit-filter` exigem o `nano` instalado no `PATH`.

## Configuração

Depois de `dolly init`, edite:

- `~/.config/dolly/paths.conf` — pares `source|destination`, um por linha.
- `~/.config/dolly/filters.txt` — regras de exclusão no formato
  [`--filter-from`](https://rclone.org/filtering/) do rclone (ex: `- .git/**`).

## Logs

Cada execução grava seus logs em:

```text
~/.local/share/dolly/logs/dry-run/<timestamp>/
~/.local/share/dolly/logs/prod/<timestamp>/
```

Dentro de cada pasta, por par sincronizado:

- `<job>.rclone.log` — saída técnica detalhada do rclone.
- `<job>.changes.log` — lista estilo git status (`+` adicionado, `*`
  modificado, `-` removido). A linha `-` só aparece em execuções com `--mirror`.

E um `summary.log` com o resumo agregado de todos os pares da execução.
