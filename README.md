## Instruções

O hook é ativado a cada push realizado e os commits são analisados por suspeitas de credenciais utilizando uma ferramenta chamada TruffleHog.

O hook irá barrar o push se identificar uma credencial em potencial e apresentará um relatório na forma [hash] trecho suspeito, por exemplo como a seguir:

```
Commit 1b148697562b (20/07/2020) Branch: SDI-2222
[584b8f00] /src/main/java/objective/ArquivoComCredencial.java:  "password=p4s5w0rd"
[771a4e3d] /src/main/java/objective/OutroArquivoComCredencial.java:  "public String mensagem="Informe sua senha:"
```

Para resolver o problema e o hook aceitar o push, há duas opções:

1) Se é realmente uma credencial, remova!
2) Se você identificar que foi um falso positivo, pode marcar como falso positivo na mensagem do commit em que foi encontrado o falso positivo (não necessariamente é o último commit).

Para informar que uma linha não contém uma credencial de fato, você deverá alterar o commit (usando amend ou algum processo que altere o histórico, como squash)
na mensagem de commit os hashes correspondentes aos trechos que são falsos positivos, através da palavra "FalsoPositivo:" seguida dos hashes. Por exemplo:

```
[CHAVE-42] Meu commit
FalsoPositivo: 771a4e3d
```

Você pode informar mais de um hash separado por vírgula. Ao retentar o push, você será apresentado com um novo relatório:

```
Commit 1b148697562b (20/07/2020) Branch: SDI-2222
[771a4e3d] [falso +] /src/main/java/objective/OutroArquivoComCredencial.java:  "public String mensagem="Informe sua senha:"
```

Observe que a linha marcada estará com a tag "falso +" e o commit será aceito.

No relatório informado, há algumas sugestões de como corrigir seu commit para que ele seja aceito pelo hook:

- Squash dos commits afetados e identificação do falso positivo no commit de squash: `git merge --squash <commit>`
- Rebase interativo: `git rebase -i <commit>`

Caso haja alguma dificuldade, dúvidas ou sugestões com relação ao procedimento, estamos à disposição.


### Problemas comuns

1) Coloquei os “FalsoPositivo”s na mensagem de commit mas continua dando o mesmo erro!

   Verifique se a mensagem “FalsoPositivo: HASH1,HASH2,...” está em sua própria linha e na mensagem do commit afetado! Lembre-se que o git trabalha com uma sequência de diffs e a informação que está causando o falso positivo pode não estar no último commit antes do push, atente-se ao hash do commit indicado logo antes da mensagem da credencial em vermelho.

   Você pode adicionar mais linhas a um commit usando múltiplas flags -m no comando git commit ou fazendo a mensagem de commit (ou amend) diretamente em um editor de texto. Você pode também usar o modo interativo de rebase, que permitirá escolher de quais commits quer editar a mensagem.


2) Estou tentando fazer o merge do meu Merge Request no Gitlab e dá erro de credenciais encontradas, mas coloquei os falsos positivos em todos os commits e fiz push com sucesso até agora.

   Se você está fazendo um merge com squash de commits, as mensagens com falsos positivos serão perdidas. Olhe seus commits com falsos positivos (em uma MR é fácil, basta olhar na lista de commits no Gitlab os que tem “...” do lado da mensagem) e adicione os mesmos na mensagem de commit de squash.


3) Coloquei credenciais sem querer no meu código e tirei no outro commit, mas continua dando erro. Como faço?

   Fazer um commit tirando as credenciais do código não as retira do histórico. Analise: você consegue ver a credencial olhando o commit que você fez para tirar ela.

   Como resolver? Bem, se você não se importa de perder as mensagens de commit, você pode fazer squash. Isso removerá do histórico a credencial, já que o squash terá apenas a diferença entre o estado anterior e a atual, sem as credenciais.

   Se você não quer perder as mensagens de commit, use uma ferramenta como o BFG Repo Cleaner para expurgar as credenciais do histórico.
