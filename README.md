# DPOC Clínico

Ferramenta de apoio à decisão para o manejo da DPOC segundo a estratégia GOLD, em
formato *wizard* (uma pergunta ou decisão por tela, com barra de progresso e botão
de voltar).

Página única, sem backend e sem banco de dados: **`index.html`** com HTML, CSS e
JavaScript embutidos. Basta abrir o arquivo no navegador.

## As 9 etapas

| # | Etapa | O que faz |
|---|-------|-----------|
| 1 | Avaliação inicial | Nome, idade, tabagismo com cálculo automático da carga tabágica em maços-ano, escala mMRC, CAT (8 itens, soma automática 0–40), VEF₁ pós-broncodilatador e exacerbações nos últimos 12 meses |
| 2 | Classificação | Resumo consolidado + grau espirométrico (GOLD 1–4) e grupo ABE, com a justificativa de cada um |
| 3 | Objetivos terapêuticos | Reduzir sintomas e reduzir risco futuro, com destaque do que é prioritário no grupo do paciente |
| 4 | Tratamento inicial | Esquema por grupo (A / B / E), incluindo eosinófilo ≥ 300 células/µL para terapia tripla, e as medidas não farmacológicas obrigatórias |
| 5 | Reavaliação | Reaplica CAT e mMRC, registra exacerbações do intervalo, esquema em uso, adesão e técnica inalatória |
| 6 | Resposta ao tratamento | Compara com a consulta anterior e classifica a resposta em duas dimensões independentes: dispneia e exacerbações |
| 7 | Ajuste do tratamento | Árvore de seguimento do GOLD, ramificando pelo problema predominante (dispneia, exacerbação ou ambos) |
| 8 | Exacerbação aguda | Acessível a qualquer momento pelo cabeçalho: gravidade, critério de Anthonisen para antibiótico, corticoide oral por 5 dias e critérios de hospitalização |
| 9 | Acompanhamento | Prazo de retorno e checklist da próxima consulta; "Registrar retorno" reabre a Etapa 5 com os dados já carregados |

## Regras clínicas implementadas

- **Grau espirométrico:** GOLD 1 ≥ 80% · GOLD 2 = 50–79% · GOLD 3 = 30–49% · GOLD 4 < 30%.
- **Grupo ABE:** E se ≥ 2 exacerbações moderadas **ou** ≥ 1 grave no último ano (prevalece
  sobre A/B); caso contrário, B se mMRC ≥ 2 **ou** CAT ≥ 10, senão A.
- **Resposta inadequada na dispneia:** paciente ainda sintomático (mMRC ≥ 2 ou CAT ≥ 10)
  sem melhora clinicamente relevante — queda do CAT < 2 pontos (diferença mínima
  clinicamente importante) e mMRC sem redução.
- **Resposta inadequada nas exacerbações:** qualquer exacerbação moderada ou grave no
  intervalo. Quando as duas dimensões estão inadequadas, segue-se o braço de exacerbações.
- **Antibiótico na exacerbação:** aumento da purulência associado a aumento do volume
  e/ou piora da dispneia, ou os três sintomas cardinais de Anthonisen.

## Dados

O estado do paciente fica em memória durante a sessão e é persistido em `localStorage`
do próprio navegador (chave `dpoc_clinico_v1`) — nada é enviado a servidores. A tela
inicial permite reabrir o paciente, revisar a avaliação basal, registrar um novo retorno
ou apagar os dados.

## Aviso

Ferramenta educacional de apoio à decisão — não substitui o julgamento clínico.
As referências completas, em ABNT, estão no rodapé da própria página.
