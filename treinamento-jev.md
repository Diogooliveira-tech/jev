# Treinamento de JEV: do primeiro conceito aos usos em QA

Material de estudo individual baseado em `anotacoes.md` e na imagem enviada. Documentação consultada em 22/10/2026. Os exercícios e metas abaixo são propostas didáticas; não são resultados medidos nem um workshop oficial da TypeSafe.

## 1. O que é JEV?

Neste contexto, **Jev é um modelo de IA da TypeSafe AI especializado em decisões estruturadas**. Você fornece informações sobre uma situação e perguntas com formatos definidos. Ele devolve valores que seu programa consegue usar para escolher um caminho, organizar uma fila ou pedir revisão.

Exemplo: diante de um relato sobre um pagamento, seu programa pode perguntar qual equipe deve investigar. Em vez de receber um parágrafo, recebe uma opção dentre as equipes permitidas.

A TypeSafe chama essa abordagem de **System One**, voltada a julgamentos rápidos e delimitados. Perguntas que exigem investigação extensa devem ser divididas ou encaminhadas a outro processo. [Introdução oficial](https://docs.typesafe.ai/introduction).

Pense em quatro etapas:

```text
Contexto observado → pergunta definida → resposta estruturada → ação do programa
```

**Resposta com formato correto ainda pode estar errada.** A qualidade depende também das evidências, dos critérios e da avaliação no seu produto.

### Onde cada ferramenta entra?

| Recurso | Papel no treinamento | Exemplo |
|---|---|---|
| Regra de código | Verificar algo exato | Comparar total esperado e total observado |
| Jev | Julgar algo dentro de opções e critérios definidos | Classificar uma observação como suspeita ou inconclusiva |
| Modelo generativo | Produzir explicações e conteúdo | Escrever uma explicação sobre uma falha |
| Playwright | Interagir com o navegador e verificar resultados | Preencher um formulário e conferir uma mensagem |
| Pessoa responsável | Definir critérios e resolver dúvidas | Confirmar se o comportamento viola uma regra do produto |

Essa distribuição é a arquitetura proposta para os exercícios. Modelos generativos também podem produzir dados estruturados; a distinção aqui é a finalidade específica de Jev.

Uma correção nas suas anotações: **LLM significa Large Language Model, ou modelo de linguagem de grande porte**. A expressão “mestrado em Direito” é um erro de tradução neste contexto.

## 2. Como estudar

**Carga sugerida: 8 horas**, em oito sessões de uma hora. É uma trilha introdutória mais gradual que o workshop de duas horas descrito nas anotações.

Você pode começar sem instalar ferramentas ou contratar uma API. Para a prática conectada, precisará de acesso autorizado à API da TypeSafe e de um ambiente de demonstração com Playwright. Credenciais devem ficar fora dos arquivos versionados.

| Sessão | Tema | Entrega |
|---|---|---|
| 1 | Fundamentos e escolha da ferramenta | Cinco decisões possíveis no seu trabalho |
| 2 | Choice, Score e Noul | Três perguntas com critérios claros |
| 3 | Contexto e incerteza | Uma política de encaminhamento |
| 4 | Primeira requisição | Um JSON de entrada e análise da resposta |
| 5 | Projeto 1: Jev + Playwright | Desenho de um fluxo de observação e julgamento |
| 6 | Projeto 2: triagem de código | Fila de achados priorizada e revisada |
| 7 | Projeto 3: observabilidade | Planilha ou tabela de avaliação |
| 8 | Avaliação final | Comparação entre processo atual e proposta com Jev |

Em cada sessão: 15 minutos de leitura, 35 de exercício e 10 de revisão.

## 3. Aula 1 — Reconhecer uma boa decisão para Jev

Uma boa pergunta tem um objetivo pequeno e contexto suficiente. “Qual equipe atende este relato?” é delimitada. “Investigue todo o sistema e resolva o incidente” reúne muitas tarefas.

Antes de usar IA, pergunte: existe uma regra exata que resolve isso? Para conferir uma soma ou validar um campo obrigatório, comece com código. Considere Jev quando houver interpretação de linguagem ou classificação contextual.

**Exercício:** liste cinco decisões repetidas no seu trabalho. Para cada uma, escreva a informação necessária, as respostas permitidas e a consequência de um erro. Escolha uma decisão para automatizar e explique por quê.

**Critério de conclusão:** você consegue separar coleta de evidências, julgamento e execução da ação.

## 4. Aula 2 — As três primitivas

“Primitiva” significa um bloco básico com o qual você monta uma solução.

### Choice: escolher uma opção

Use para categorias: `produto`, `ambiente`, `automacao` ou `inconclusivo`. Descreva o que distingue cada alternativa. A resposta inclui a escolha, probabilidades por alternativa e `confidence`. Inclua uma saída como `outro` quando as categorias não cobrirem todas as situações. [Choice oficial](https://docs.typesafe.ai/primitives/choice).

### Score: avaliar em uma escala

Use para uma graduação com níveis descritos. Exemplo de impacto: 0 = visual; 1 = função prejudicada com alternativa; 2 = tarefa essencial bloqueada. A resposta pode ser fracionária: `1.6` representa uma posição na escala, não uma probabilidade de 160%. [Score oficial](https://docs.typesafe.ai/primitives/score).

### Noul: avaliar uma afirmação

Use para uma pergunta de sim ou não, como “O relato descreve perda de dados?”. O campo `noul` varia entre 0 e 1 e expressa a probabilidade de sim. Um valor de `0.5` indica incerteza, não “metade de uma perda de dados”. Não há campo separado `confidence` nessa resposta. [Noul oficial](https://docs.typesafe.ai/primitives/noul).

**Exercício:** para a observação “Após salvar, o formulário fica vazio e o registro não aparece na busca”, escreva uma pergunta de cada tipo. Não invente se o registro foi perdido: uma busca vazia, isoladamente, pode não provar isso.

**Gabarito orientativo:** Choice para encaminhamento; Score para impacto com níveis explícitos; Noul para presença de uma evidência específica, como “A observação informa que os campos ficaram vazios após salvar?”.

## 5. Aula 3 — Contexto e incerteza

O **state** é a informação fornecida ao modelo: requisito, observação, logs relevantes e condições da execução. Para este treinamento, use registros com estes campos:

```json
{
  "requisito": "Depois de salvar, o registro deve aparecer na lista.",
  "observacao": "A mensagem de sucesso apareceu, mas a lista permaneceu vazia.",
  "condicoes": "Busca sem filtros; atualização da lista ainda não verificada.",
  "evidencia": "Texto coletado da página e horário da tentativa."
}
```

Em Choice e Score, `confidence` resume a concentração das probabilidades. **Não significa diretamente a taxa de acerto observada no seu produto.** Um modelo pode estar confiante e errado. Limiares dependem das consequências e precisam ser avaliados com exemplos conhecidos. [Confiança oficial](https://docs.typesafe.ai/confidence).

**Política didática para uma classificação de baixo impacto:**

| Condição | Encaminhamento proposto |
|---|---|
| Evidência insuficiente ou contraditória | Coletar mais dados ou pedir revisão |
| Categoria inconclusiva | Pedir revisão |
| Confidence abaixo de 0.90 | Pedir revisão |
| Confidence a partir de 0.90 | Sugerir uma categoria e registrar a decisão |

O valor 0.90 é um ponto de partida fictício, não uma recomendação universal. Essa tabela trata de Choice; não aplique o campo `confidence` a Noul.

**Exercício:** pegue três observações e identifique qual informação falta para julgar cada uma. Depois, explique o que fazer se o modelo não responder ou a requisição falhar. Sugestão: registrar como não avaliado e preservar a evidência.

## 6. Aula 4 — Entender uma requisição

O exemplo abaixo é um corpo JSON didático baseado no contrato documentado de Choice. Não é um programa completo nem foi enviado à API neste treinamento.

```json
{
  "model": "jev-latest",
  "state": {
    "requisito": "Salvar deve incluir o registro na lista.",
    "observacao": "Após salvar, apareceu sucesso, mas a lista está vazia.",
    "limitacao": "Não sabemos se a lista terminou de atualizar."
  },
  "questions": {
    "triagem": {
      "type": "choice",
      "instructions": "Como encaminhar a observação em relação ao requisito, considerando a limitação informada?",
      "criteria": {
        "compativel": "Há evidência suficiente de que o requisito foi atendido.",
        "suspeito": "Há evidência suficiente de uma possível violação a investigar.",
        "inconclusivo": "Faltam informações para avaliar o atendimento ao requisito."
      }
    }
  }
}
```

O programa lê a resposta em `answers.triagem`, verifica os campos e aplica sua política. Guarde também a versão efetivamente retornada pelo serviço. O alias `jev-latest` pode evoluir; para comparar experimentos, registre a versão e mantenha as demais condições constantes.

Perguntas na mesma chamada usam o mesmo estado e são independentes. Se uma segunda pergunta precisar da resposta da primeira, essa dependência deve ser organizada pelo código. [Referência de perguntas](https://docs.typesafe.ai/primitives).

**Exercício sem API:** explique cada campo do JSON e proponha como obter a informação sobre a atualização da lista.

**Prática opcional com API:** siga a documentação oficial de acesso, envie apenas um exemplo sintético e registre resposta, duração e versão. Compare a decisão com sua avaliação prévia. Não confunda uma resposta bem formada com uma classificação correta.

## 7. Projeto 1 — Jev com Playwright

**Objetivo:** usar julgamento estruturado para encaminhar observações de uma aplicação de demonstração.

Fluxo proposto:

```text
Playwright executa uma interação
  → coleta texto e resultado observado
  → verificações exatas conferem o que pode ser medido
  → Jev classifica a observação contextual
  → código aplica a política de revisão
  → evidências ficam registradas
```

No projeto, Jev recebe uma representação textual da observação. Uma captura guardada como evidência para uma pessoa não é, por si só, uma entrada visual para o modelo.

**Atividade:** escolha um formulário de demonstração e documente a regra de sucesso. Defina quais dados o Playwright deve coletar, escreva uma pergunta Choice e desenhe os caminhos `compativel`, `suspeito` e `inconclusivo`. Mantenha as verificações exatas do resultado no código.

**Entrega:** um registro contendo requisito, ação, observação, verificação exata, julgamento do modelo e encaminhamento. Se não houver API, preencha o julgamento como “simulado”.

**Critério de conclusão:** é possível explicar a decisão usando as evidências registradas; uma classificação “suspeito” gera investigação, sem virar automaticamente um defeito confirmado.

## 8. Projeto 2 — Triagem de saúde do código

**Objetivo:** organizar achados para que uma pessoa investigue os mais relevantes primeiro.

Use cinco achados sintéticos ou exportados de uma ferramenta de análise. RepoWise e CodeScene são exemplos mencionados nas suas anotações; este exercício não pressupõe integração pronta com eles.

Para cada achado, reúna descrição, módulo, trecho relevante, frequência de alteração e impacto conhecido. Marque dados ausentes explicitamente.

**Atividade:** proponha um Choice para encaminhar o achado à equipe responsável e dois Scores separados para impacto e urgência. Descreva cada nível. Combine as avaliações com uma regra sua e revise manualmente a ordem final.

Exemplo de regra didática, para duas escalas de 0 a 2:

```text
prioridade = 0.7 × impacto + 0.3 × urgência
```

Os pesos são hipóteses de negócio. Antes de ordenar, se faltar contexto essencial ou a confiança for baixa, envie à revisão. Um score não prova que existe defeito ou vulnerabilidade.

**Entrega:** tabela com achado, contexto, classificações, prioridade sugerida e decisão humana. Explique pelo menos uma discordância.

## 9. Projeto 3 — Medir se houve benefício

**Objetivo:** comparar o processo atual com o processo assistido por Jev.

Comece com 20 exemplos rotulados por você, incluindo situações claras, ambíguas e com informação ausente. Use dez para ajustar perguntas e dez separados para uma primeira avaliação. Essa amostra serve para aprender; não demonstra confiabilidade em produção.

| Campo por exemplo | O que registrar |
|---|---|
| Referência humana | Categoria esperada e motivo |
| Modelo | Versão, pergunta e resposta |
| Acerto | Se a categoria coincidiu com a referência |
| Encaminhamento | Automático, revisão ou não avaliado |
| Tempo | Duração da chamada e duração total do processo |
| Consumo | Tokens e custo informado ou calculado com tarifa identificada |
| Revisão | Tempo humano e correção realizada |

Calcule:

```text
Taxa de acerto = classificações corretas / classificações avaliadas
Cobertura automática = decisões sem revisão / total de exemplos
Erro entre automáticas = decisões automáticas incorretas / decisões automáticas
Custo por acerto = custo total medido / classificações corretas
```

Se o denominador for zero, registre “não aplicável”. Registre falhas da API separadamente. Meça tokens, custos e duração por instrumentação e logs; uma estimativa produzida pelo próprio modelo não substitui essas medições.

**Atividade:** compare o mesmo conjunto com classificação humana e, quando possível, uma regra simples. Inclua revisão e retrabalho no tempo total. Teste limiares diferentes e observe se reduzir erros aumenta a fila de revisão.

**Entrega:** uma conclusão sustentada pelos dados: em quais situações Jev ajudou, quais continuam inconclusivas e qual seria o próximo experimento.

## 10. Avaliação final

Responda sem consultar o material:

1. Qual a diferença entre o estado e a pergunta?
2. Quando escolher Choice, Score ou Noul?
3. Noul de 0.5 significa impacto médio?
4. Confidence de 0.95 comprova 95% de acerto no seu produto?
5. Quem executa a interação com o navegador no projeto 1?
6. Como lidar com contexto insuficiente?
7. Por que separar exemplos de ajuste e avaliação?
8. Como saber se a solução economizou trabalho?

**Gabarito resumido:**

1. Estado é a informação avaliada; pergunta define o julgamento desejado.
2. Categoria; posição em escala; probabilidade de uma afirmação de sim/não.
3. Não, indica incerteza sobre a afirmação.
4. Não, é necessário medir acerto com referência no seu contexto.
5. Playwright; o código coordena execução e verificações.
6. Coletar dados ou encaminhar à revisão, preservando a evidência.
7. Para avaliar generalização e reduzir ajuste aos próprios exemplos.
8. Comparar qualidade, tempo total, custo, revisão e retrabalho com uma referência anterior.

Considere a introdução concluída quando conseguir explicar essas respostas e entregar os três exercícios de projeto, distinguindo resultados simulados dos medidos.

## 11. Seu primeiro passo hoje

Estude as aulas 1 e 2. Depois complete:

> “No meu trabalho, quero usar Jev para decidir ______, com base em ______. As respostas permitidas serão ______. Se faltarem informações, o processo vai ______.”

Não precisa instalar nada para fazer essa primeira atividade.

## Fontes

- [Introdução da TypeSafe](https://docs.typesafe.ai/introduction)
- [Perguntas e composição](https://docs.typesafe.ai/primitives)
- [Choice](https://docs.typesafe.ai/primitives/choice)
- [Score](https://docs.typesafe.ai/primitives/score)
- [Noul](https://docs.typesafe.ai/primitives/noul)
- [Confidence](https://docs.typesafe.ai/confidence)
- Material fornecido: `anotacoes.md` e imagem dos três projetos.
