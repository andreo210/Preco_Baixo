---
name: documentacao-modelagem
description: Gera documentação de modelagem de sistema — casos de uso, MER (conceitual), DER (lógico), diagrama de classes UML, diagrama de estados, diagramas de sequência e dicionário de dados — toda em Mermaid, legível direto no GitHub/GitLab sem plugin. Funciona em qualquer linguagem, framework ou domínio: explora o código real do projeto-alvo (ou entrevista o usuário quando ainda não há código) para que os diagramas usem nomes, campos e convenções reais em vez de placeholders genéricos. Use sempre que o usuário pedir para documentar a arquitetura/modelagem/domínio de um sistema, criar diagrama de classes, MER, DER, modelo entidade-relacionamento, dicionário de dados, caso de uso, diagrama de sequência, diagrama de estados, "documentação técnica" ou "UML" do projeto — mesmo que ele não use exatamente essas palavras, por exemplo "documenta esse sistema pra quem for mexer depois" ou "quero uns diagramas pra colocar no README".
---

# Documentação de modelagem

Sete tipos de artefato, todos em Mermaid (renderiza nativo no GitHub/GitLab — zero fricção para quem só quer abrir o `.md` e ver o diagrama). O valor real desta skill não é "sei a sintaxe do Mermaid" — é o processo antes de desenhar: **descobrir o terreno real do projeto-alvo antes de inventar qualquer nome de campo, método ou regra.** Um diagrama de classes com `GetSaldo()` quando o método real se chama `ObterSaldo()` é pior que não ter diagrama — ele ensina o próximo dev a procurar a coisa errada.

## Passo 1 — Descubra o terreno antes de desenhar

**Se já existe código no projeto-alvo:** explore antes de escrever qualquer diagrama. Procure especificamente por:
- Como uma entidade/modelo típica é nomeada e estruturada (convenção de chave primária, como campos obrigatórios/opcionais aparecem, se há uma entidade "rica" com métodos ou é um DTO anêmico).
- Como multi-tenancy/dono do dado é resolvido, se existir (campo de usuário/tenant, filtro global de query).
- O padrão de camada de aplicação: onde regra de negócio mora, como erro de validação é comunicado (exceção? objeto de notificação acumulado? `Result<T>`?), qual o nome real do método que persiste/atualiza/consulta.
- **Se esse mecanismo de erro é usado de forma consistente em todos os fluxos, ou se há caminho que escapa dele.** É comum um projeto ter um padrão "oficial" (uma exceção de domínio, um objeto de notificação) e, em algum canto, um caminho que lança algo genérico demais e vaza como erro 500/não tratado em vez do 4xx esperado. Vale a pena checar isso ao explorar — quando aparecer, é exatamente o tipo de achado que vira uma nota de zona cinzenta valiosa num diagrama de sequência (ver Passo 3), não algo para silenciosamente ignorar ou "consertar" na documentação.
- Se há uma transação explícita para operações que precisam ser atômicas, e qual o nome real dela.
- Se já existe uma skill, README de arquitetura, ou documento de regras de negócio no repositório — trate como fonte da verdade para as restrições que os diagramas vão modelar; não reinvente uma regra que já está escrita em outro lugar, referencie-a.

Use `grep`/exploração de arquivo real, não assuma por convenção de mercado. Os nomes de método usados nos diagramas de sequência, por exemplo, devem ser os nomes reais do repositório de dados do projeto — se você não sabe o nome real, é sinal de que ainda não explorou o suficiente. A mesma disciplina vale para restrição de dado: só marque algo como único/obrigatório/validado (`UK`, "obrigatório", etc.) quando o código de fato impõe isso — um campo que *parece* que deveria ser único pela convenção do domínio (email, CPF, ISBN) mas não tem nenhuma checagem/constraint no código é uma inferência, não um fato, e vira nota ("esperado pelo domínio, não confirmado no código") em vez de uma marcação afirmativa.

**Se ainda não existe código (fase de planejamento):** não tem o que explorar — então pergunte. Antes de desenhar qualquer diagrama, entreviste o usuário com perguntas objetivas (a ferramenta de pergunta de múltipla escolha, quando disponível, funciona bem aqui — 2-4 opções concretas por pergunta, não um campo aberto) sobre as decisões que mudariam o diagrama:
- Quais são as entidades/conceitos principais e como se relacionam?
- Para cada restrição não óbvia: o comportamento é "bloquear" ou "avisar sem bloquear"? (isso muda se a regra vira uma seta de `«extend»` condicional num caso de uso, um `alt` de bloqueio num diagrama de sequência, ou nem aparece)
- Alguma entidade tem um ciclo de vida real (mais de dois estados, com transições restritas)?
- O que está fora do escopo por enquanto? (documente isso explicitamente — ver Passo 4)

Não avance para desenhar sem essas respostas. Um diagrama bonito baseado numa regra inventada é pior que nenhum diagrama — alguém vai confiar nele.

## Passo 2 — Decida quais dos 7 artefatos valem a pena

Nem todo projeto precisa dos sete. Gerar os sete sempre, mesmo quando um deles não tem conteúdo de verdade, produz documentação que ninguém lê porque aprendeu a não confiar nela. Use este critério por artefato — detalhe de cada um, com exemplos de sintaxe, em `references/guia-por-artefato.md`:

| Artefato | Vale a pena quando... | Pule quando... |
|---|---|---|
| Casos de uso | Quase sempre — é a visão funcional mais barata de produzir | O sistema é uma biblioteca/lib interna sem "usuário" que interage |
| MER (conceitual) | Há dados persistidos e mais de ~3 entidades relacionadas | Sistema trivial de 1-2 tabelas — vá direto pro DER |
| DER (lógico) | Alguém vai implementar/alterar schema a partir daqui | Só o MER já responde a pergunta que motivou o pedido |
| Diagrama de classes | Há (ou vai haver) objetos com comportamento/regra, não só campos | Tudo é DTO anêmico/CRUD puro — o DER já cobre |
| Diagrama de estados | Uma entidade tem 3+ estados com transição restrita (nem toda válida) | O "estado" é só um `bool ativo/inativo` — isso é um filtro, não uma máquina de estados |
| Diagramas de sequência | Fluxos específicos são propensos a erro ou amarrados a uma regra crítica | Cobertura exaustiva de todo endpoint — isso é ruído, não documentação. Escolha 3-6 fluxos, não 30 |
| Dicionário de dados | Quase sempre — é a referência de consulta mais usada no dia a dia | Redundante com um DER já bem anotado num sistema minúsculo |

Diga ao usuário quais artefatos você decidiu gerar e por quê antes de começar a escrever — é barato confirmar escopo antes, caro refazer sete arquivos depois.

## Passo 3 — Gere os artefatos

Estrutura de saída recomendada — um diretório (`docs/modelagem/` é um bom padrão, mas siga a convenção do repo se já houver uma pasta de docs): um arquivo por artefato, mais um `README.md` índice com tabela (documento / o que mostra / quando abrir) e uma ordem de leitura sugerida (funcional → dados → comportamento → referência é a ordem que funcionou bem: casos de uso → MER → DER → classes → estados → sequência → dicionário).

Para a sintaxe Mermaid específica de cada tipo de diagrama (incluindo os detalhes que quebram silenciosamente se você escrever do jeito "óbvio mas errado" — comentário sem aspas em `erDiagram`, newline real em vez de `\n` literal numa `Note` de `sequenceDiagram`, etc.), leia **`references/sintaxe-mermaid.md`** antes de escrever o primeiro bloco de código de cada tipo que você ainda não usou nesta sessão.

Para orientação de conteúdo por artefato (o que incluir num MER que não faz sentido num DER, como decidir `«include»` vs `«extend»` num caso de uso, como diferenciar composição de associação num diagrama de classes, como não duplicar estado persistido com estado computado), leia **`references/guia-por-artefato.md`**.

Para um exemplo completo e real de como isso fica no fim — um domínio financeiro modelado em .NET com os 7 artefatos completos, incluindo as notas de "por quê" que fazem a diferença entre documentação que alguém lê e documentação que só existe — veja **`references/exemplo-financeiro/`**. É um exemplo concreto para calibrar nível de detalhe e tom, não um template para copiar literalmente: se o projeto-alvo for Python/Django ou TypeScript/NestJS, o diagrama de classes e os diagramas de sequência devem refletir os métodos e padrões *daquele* projeto, não os nomes em português/.NET do exemplo.

### Convenções de qualidade que valem para todos os sete

- **Explique o porquê, não só o quê.** Uma restrição sem a razão por trás ("saldo não pode ser negativo") convida alguém a "consertar" o que parece um bug assim que o requisito mudar de contexto. Uma frase de motivo evita isso.
- **Marque zona cinzenta explicitamente.** Quando uma decisão não foi confirmada (pelo código ou pelo usuário), escreva algo como "extensão — confirme antes de implementar" em vez de inventar um comportamento plausível. Isso vale tanto para o campo `Fora do escopo` do índice quanto para notas inline nos diagramas.
- **Toda transição/relação ausente é uma proibição implícita**, principalmente em diagrama de estados e de classes — se não tem seta, não é permitido. Isso só funciona se o diagrama for de fato completo para o que ele cobre; não deixe "buracos" que pareçam omissão acidental.
- **Adapte-se ao projeto-alvo, não ao seu exemplo mental.** Se o projeto usa `snake_case` e Python, o diagrama de classes usa `snake_case` e reflete os padrões Python reais (dataclass? Pydantic? SQLAlchemy declarative?) — não a convenção C# do `references/exemplo-financeiro/`.

## Passo 4 — Feche com o que ficou de fora

Todo `README.md` índice termina com uma seção curta de escopo explícito: o que foi coberto e — igualmente importante — o que ficou de fora por ser extensão/zona cinzenta não confirmada. Isso é o que impede a documentação de ser lida como "está tudo decidido" quando na verdade parte foi deixada em aberto de propósito.
