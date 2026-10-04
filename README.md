# Configuração do Ly

Este repositório acompanha a configuração local do **Ly**, o display manager em modo texto instalado nesta máquina. O diretório de trabalho é `/etc/ly`; `~/.config/ly-dm` é apenas um link simbólico para ele, mantido pelo repositório principal de dotfiles.

## Escopo

O repositório versiona os arquivos presentes em `/etc/ly`, inclusive a configuração principal, textos de idioma, scripts de inicialização, sessões personalizadas e os recursos `.dur`. Ele não contém o executável `ly-dm`, PAM, unidades systemd fornecidas pelo pacote, fontes nem credenciais.

O serviço instalado pelo pacote inicia o programa com:

```text
/usr/bin/ly-dm
```

A unidade é `ly@.service`; consulte-a com `systemctl cat ly@.service` antes de alterar a forma de inicialização.

## Configuração ativa

Em comparação com `config.ini.example`, `config.ini` configura:

- animação `dur_file`, usando `/home/joaob/.config/ly-dur/blackhole-smooth.dur`;
- interface em português brasileiro (`lang = pt_BR`);
- limpeza do campo de senha após falha;
- animação especial somente após 100 tentativas de autenticação malsucedidas;
- ocultação de dicas de energia e da versão do Ly.

O arquivo da animação fica no repositório de dotfiles, em `.config/ly-dur/`; confirme que o link/caminho existe antes de ativar o Ly. Para outra conta de usuário, adapte o caminho em `dur_file_path`.

## Estrutura

| Caminho | Finalidade |
| --- | --- |
| `config.ini` | Configuração ativa do Ly. |
| `config.ini.example` | Referência instalada pelo pacote. |
| `lang/` | Traduções disponíveis, inclusive `pt_BR.ini`. |
| `custom-sessions/` | Arquivos desktop para sessões adicionais. |
| `startup.sh` | Gancho executado antes de o Ly assumir o TTY. |
| `setup.sh` | Prepara o ambiente da sessão após login. |
| `example.dur` | Recurso de animação de exemplo. |
| `save.txt` | Estado salvo pelo Ly; revise antes de publicar se não quiser expor nomes de conta ou seleção de sessão. |

## Instalação e restauração

Este é um diretório de sistema: altere os arquivos com privilégios administrativos e mantenha um backup antes de substituir `config.ini`.

```bash
sudo cp -a /etc/ly "/etc/ly-backup-$(date +%Y%m%d-%H%M%S)"
```

Depois de alterar a configuração, valide o conteúdo e reinicie apenas quando houver uma sessão alternativa disponível. Uma configuração inválida no display manager pode impedir o login gráfico/textual esperado.

```bash
sudo systemctl restart ly@tty2.service
```

O TTY acima é apenas um exemplo: confira a instância ativa com `systemctl status 'ly@*.service'` antes de reiniciar ou habilitar qualquer unidade.

## Git e sincronização

O repositório usa a branch `main` e o remoto `personal`:

```text
git@github.com:joaobertholino/ly-dm.git
```

O repositório de dotfiles executa `~/.local/bin/auto-push.sh` por meio do timer de usuário `git-auto-push.timer`. O script registra alterações deste diretório e envia `main` ao remoto `personal` quando ele está configurado.

Também é possível sincronizar manualmente:

```bash
git -C /etc/ly status
git -C /etc/ly add -A
git -C /etc/ly commit -m "chore: update Ly configuration"
git -C /etc/ly push personal main
```

Os metadados Git em `/etc/ly/.git` pertencem ao usuário `joaob`, enquanto os arquivos de configuração continuam pertencendo a root. Isso permite ao serviço de usuário registrar alterações legíveis sem conceder escrita nas configurações do sistema. Use `sudo` para editar os arquivos em `/etc/ly`.

## Segurança

- Nunca coloque senhas, chaves privadas, tokens ou dados de PAM neste repositório.
- Revise `config.ini` cuidadosamente antes de alterar opções de login automático, usuários permitidos ou autenticação.
- `save.txt` contém estado do Ly; mantenha-o fora do histórico se passar a conter informação que não deva ser publicada.
- Teste mudanças a partir de uma sessão já autenticada e mantenha uma forma de recuperação por TTY.
