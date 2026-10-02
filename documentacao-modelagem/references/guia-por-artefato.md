# Guia de conteúdo por artefato

Enquanto `sintaxe-mermaid.md` cobre "como não quebrar o bloco de código", este documento cobre "o que colocar dentro dele" — o critério de conteúdo que separa cada um dos sete artefatos do vizinho mais próximo (MER de DER, caso de uso de diagrama de sequência).

Índice: [Casos de uso](#casos-de-uso) · [MER](#mer-conceitual) · [DER](#der-lógico) · [Diagrama de classes](#diagrama-de-classes) · [Diagrama de estados](#diagrama-de-estados) · [Diagramas de sequência](#diagramas-de-sequência) · [Dicionário de dados](#dicionário-de-dados) · [O índice (README)](#o-índice-readme)

## Casos de uso

Pergunta que o diagrama responde: "quem interage com o sistema, e o que cada um consegue fazer?" — visão funcional, zero implementação.

- Um ator por tipo de interação relevante — inclua atores não-humanos (job agendado, sistema externo que chama via webhook) quando eles disparam comportamento sozinhos, não só o usuário final.
- Um caso de uso por capacidade que o usuário reconheceria e nomearia ("Registrar despesa"), não por operação técnica ("Executar INSERT na tabela X").
- `«include»` só para o que **sempre** acontece dentro do caso de uso base — se você tiraria a seta e o caso base ainda faz sentido sozinho na maioria das vezes, é `«extend»`, não `«include»`. Errar essa escolha é o erro conceitual mais comum, veja exemplos em `sintaxe-mermaid.md`.
- Depois do diagrama, escreva uma seção curta explicando as relações que não são óbvias — por que X inclui Y, por que Z é `extend` e não `include`. É aqui que a decisão de "bloqueia vs. avisa" (levantada no Passo 1 da skill) aparece pela primeira vez, como pista para o resto da documentação.

## MER (conceitual)

Pergunta: "quais são as entidades e como se relacionam, sem entrar em como isso vira tabela?" — para quem quer entender o domínio sem saber ler `erDiagram` de verdade.

- Um `erDiagram` com bloco de atributos **minimalista**: 2-4 campos que ajudam a reconhecer a entidade (nome, um identificador de negócio), nunca a lista completa de colunas — isso é o DER.
- Cardinalidade correta importa mais aqui do que em qualquer outro artefato, porque é o que alguém de negócio vai realmente ler. "Um cliente pode ter zero pedidos" (`||--o{`) é uma afirmação diferente de "todo cliente tem pelo menos um pedido" (`||--|{`) — confirme qual é o caso antes de escrever.
- Feche com uma seção "leitura rápida" apontando as relações que fogem do óbvio — a mesma técnica usada nos casos de uso, aplicada a relacionamento de dados (ex.: "por que essa entidade se relaciona consigo mesma", "por que essas duas entidades não se tocam diretamente").

## DER (lógico)

Pergunta: "o que exatamente vira coluna, com que tipo e restrição, quando alguém for implementar isso?" — é o MER com bloco de atributos completo.

- Todo atributo do bloco tem tipo real (do banco/ORM do projeto-alvo, não um tipo genérico), marcação de chave (`PK`/`FK`/`UK`) e, quando não for óbvio, um comentário entre aspas explicando o domínio de valores ou a restrição.
- **`UK`/`PK`/`FK` são afirmações sobre o que o código impõe, não sobre o que "faz sentido de negócio".** Antes de marcar um campo como `UK` (email, CPF, ISBN — os candidatos óbvios), confirme que existe de fato uma constraint/validação de unicidade no código ou no schema. Se não existe, não marque — vire um comentário entre aspas tipo `"esperado como único pelo domínio, sem constraint confirmada no código"`. Marcar `UK` sem essa confirmação é o mesmo erro que inventar uma regra de negócio: parece uma afirmação de fato quando é uma suposição.
- Chave única composta, campo denormalizado, enum persistido como int vs. string — tudo isso é decisão de implementação que pertence ao DER, não ao MER. Registre a decisão **e o porquê** numa seção de "notas de mapeamento" depois do diagrama, ligando explicitamente ao ORM/framework real do projeto-alvo.
- Se um campo é derivado de outro (soma calculada, estado computado), diga isso explicitamente e aponte quem é responsável por manter a consistência (ex.: "toda escrita em X precisa recalcular Y na mesma transação") — esse é o tipo de invariante que, não documentado, vira bug silencioso em produção.

## Diagrama de classes

Pergunta: "que comportamento cada entidade tem, e quem pode chamar o quê?" — só vale a pena quando há comportamento de verdade para desenhar (ver tabela de decisão no `SKILL.md`).

- Escreva no estilo idiomático do projeto-alvo, não um estilo genérico de livro-texto de UML. Se o projeto usa uma convenção de entidade rica (factory estática, setter privado), reflita isso. Se é uma stack onde entidade é struct/dataclass anêmico e a regra mora num service separado, o diagrama de classes deve mostrar isso — inclusive a classe de serviço, se for lá que a regra de negócio realmente vive.
- Distinga agregado (raiz que controla um conjunto de entidades internas) de entidade independente. Uma entidade "tem repositório próprio" não a torna raiz de agregado sozinha — o critério é se a escrita nela precisa passar pela raiz para manter um invariante coerente (ex.: duas linhas que têm que nascer/morrer juntas).
- Depois do diagrama, uma seção "por que os métodos são esses" — cada método público existe porque alguma regra específica precisa de um lugar único para viver. Se você não consegue justificar um método com uma regra concreta, ele provavelmente não devia estar no diagrama (ou não devia existir).

## Diagrama de estados

Pergunta: "quais são todas as transições válidas desta entidade, e por implicação, quais são proibidas?" — reserve para entidades com ciclo de vida real (3+ estados com transição restrita). Um `bool ativo` não é uma máquina de estados, é um filtro — não force um `stateDiagram-v2` nele.

- Todo estado inicial começa em `[*] -->` e, se houver estado terminal de verdade, termina em `--> [*]`.
- Toda transição ausente é uma proibição implícita — releia o diagrama perguntando "existe algum caminho no código/regra que eu não desenhei?" antes de considerá-lo pronto.
- Diferencie estado **persistido** (uma coluna/campo que guarda o valor) de estado **computado** (derivado de outro campo, ex.: "atrasado" quando `data_vencimento < hoje` e ainda não pago) — computado não deve virar uma coluna nova só porque ficou mais fácil de desenhar. Anote isso explicitamente numa nota do diagrama quando for o caso.
- Uma nota por estado explicando a regra que só se aplica ali (ex.: "só neste estado o registro aceita edição") economiza uma seção de prosa inteira.

## Diagramas de sequência

Pergunta: "nesse fluxo específico, o que chama o quê, em que ordem, e o que acontece quando dá errado?" — escolha 3-6 fluxos que são propensos a erro ou amarrados a uma regra crítica, não cobertura exaustiva de endpoint. Um diagrama de sequência para cada CRUD trivial é ruído; um para "o fluxo onde dinheiro/dado sensível pode se perder se a ordem das chamadas estiver errada" é valioso.

- Participantes = as camadas/componentes reais do projeto-alvo (controller, service, repositório, fila, job) com os nomes reais de classe quando existirem.
- Mensagens usam o nome real do método (`->>Servico: NomeRealDoMetodo(...)`) — se você não sabe o nome real, volte ao Passo 1 e explore antes de continuar.
- Use `alt/else` para toda bifurcação de "isso pode ser rejeitado" — é aqui que a distinção "bloqueia vs. avisa" (decidida lá no início) fica concreta: um caminho que bloqueia tem um `alt` com um branch de erro explícito voltando pro chamador; um que só avisa não deveria usar o mesmo mecanismo de erro que um caminho que bloqueia (isso é uma armadilha de arquitetura real, não hipotética — se o framework do projeto-alvo tem um padrão único de "erro = resposta ruim", um alerta que não deveria bloquear precisa viajar por outro canal, e vale a pena documentar essa decisão explicitamente quando ela existir).
- Ao desenhar o branch de erro, confira se o fluxo realmente usa o mecanismo "oficial" de erro do projeto-alvo (a exceção/objeto de notificação identificado no Passo 1) — é comum achar um caminho que lança algo genérico demais e escapa do tratamento padrão (ex.: uma exceção que o controller não captura, e que vaza como 500 em vez do 4xx esperado). Quando encontrar isso, desenhe o `alt` como ele realmente se comporta, não como "deveria" se comportar, e registre o achado como zona cinzenta — é exatamente o tipo de inconsistência que vale mais a pena documentar do que qualquer regra que já funciona certinho.
- `Note` pontuais para justificar uma decisão não óbvia no meio do fluxo (por que abriu transação aqui, por que validou nessa ordem e não na oposta).

## Dicionário de dados

Pergunta: "o que exatamente esse campo aceita, e por quê?" — referência de consulta rápida, tabela por tabela.

- Markdown puro (tabela), não Mermaid — não existe tipo de diagrama de dicionário de dados no Mermaid, e forçar isso numa sintaxe de diagrama produziria algo pior que uma tabela simples.
- Colunas mínimas: campo, tipo, nulo?, chave, domínio/valores válidos, descrição/regra que justifica a restrição. A última coluna é a que mais gente pula e a que mais vale a pena — "por que esse campo não pode ser negativo" é mais útil que "decimal, não negativo".
- Convenções que se repetem em toda entidade (auditoria, dono do dado/tenant) — documente uma vez no topo do arquivo, não repita linha a linha em cada tabela.

## O índice (README)

O arquivo que amarra todos os outros — sem ele, sete arquivos soltos são pior que um documento só, porque ninguém sabe por onde começar.

- Tabela: documento / o que mostra / quando abrir. Isso responde "qual desses eu abro pra essa dúvida específica" sem precisar abrir todos.
- Ordem de leitura sugerida — a que funcionou bem: funcional (casos de uso) → domínio (MER) → implementação de dados (DER) → comportamento (classes → estados) → fluxo (sequência) → referência de consulta (dicionário). Ajuste se o projeto-alvo tiver uma ênfase diferente (ex.: um sistema sem UI onde caso de uso importa menos que diagrama de sequência).
- Seção explícita de **escopo coberto** e **fora do escopo** — a segunda é tão importante quanto a primeira. É o que impede a documentação de ser lida como "está tudo decidido" quando, na verdade, parte foi deixada em aberto de propósito (ver Passo 4 do `SKILL.md`).
- Se houver uma fonte de regras de negócio separada (skill, wiki, ADR), linke para ela e diga explicitamente qual documento é a fonte da verdade quando os dois divergirem.
