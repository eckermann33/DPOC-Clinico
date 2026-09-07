# DPOC Clínico

Ferramenta de apoio à decisão para o manejo da DPOC segundo a estratégia GOLD, em
formato *wizard* (uma pergunta ou decisão por tela, com barra de progresso e botão
de voltar).

Página única, sem backend e sem banco de dados: **`index.html`** com HTML, CSS e
JavaScript embutidos. Basta abrir o arquivo no navegador.

## As 9 etapas

| # | Etapa | O que faz |
|---|-------|-----------|
| 1 | Avaliação inicial | Nome, idade, tabagismo com cálculo automático da carga tabágica em maços-ano, escala mMRC, CAT (8 itens, soma automática 0–40), VEF₁ pós-broncodilatador (ou marcação de **espirometria ainda não realizada**) e exacerbações nos últimos 12 meses |
| 2 | Classificação | Resumo consolidado + grau espirométrico (GOLD 1–4) e grupo ABE, com a justificativa de cada um |
| 3 | Objetivos terapêuticos | Reduzir sintomas e reduzir risco futuro, com destaque do que é prioritário no grupo do paciente |
| 4 | Tratamento inicial | Esquema por grupo (A / B / E), incluindo eosinófilo ≥ 300 células/µL para terapia tripla, e as medidas não farmacológicas obrigatórias |
| 5 | Reavaliação | Reaplica CAT e mMRC, registra exacerbações do intervalo, esquema em uso, adesão e técnica inalatória |
| 6 | Resposta ao tratamento | Compara com a consulta anterior e classifica a resposta em duas dimensões independentes: dispneia e exacerbações |
| 7 | Ajuste do tratamento | Árvore de seguimento do GOLD, ramificando pelo problema predominante (dispneia, exacerbação ou ambos) |
| 8 | Exacerbação aguda | Acessível a qualquer momento pelo cabeçalho: gravidade, critério de Anthonisen para antibiótico, corticoide oral por 5 dias e critérios de hospitalização |
| 9 | Acompanhamento e encerramento | Prazo de retorno, checklist da próxima consulta e tela de encerramento com orientações ao paciente, prescrição e evolução para o prontuário |

## Além do wizard

- **Login por profissional.** Cada profissional entra com usuário e senha e vê apenas os
  próprios pacientes (ver *Como o login funciona*, abaixo).
- **Banco de pacientes.** Lista com busca por nome, mostrando última classificação GOLD, data
  do último atendimento e esquema em uso. Abrir um paciente vai direto para a Etapa 5
  (reavaliação) com os dados da última consulta carregados; também dá para ver o histórico de
  atendimentos e recopiar a evolução de cada um.
- **Espirometria pendente.** Se o paciente ainda não fez o exame, o grau GOLD 1–4 fica como
  "pendente" e o fluxo segue normalmente — o grupo ABE depende só de sintomas e exacerbações.
  O pedido de espirometria com prova broncodilatadora entra como aviso destacado nas
  orientações, na prescrição e no texto de prontuário.
- **Prescrição pronta para copiar.** Nas telas de conduta (Etapas 4, 7 e 8) e no encerramento,
  um bloco de texto simples com classe e princípio ativo, posologia e duração, com botão
  "Copiar prescrição".
- **Evolução de prontuário automática.** Ao encerrar o atendimento, um texto corrido é gerado
  com anamnese, escalas, classificação, comparação com a consulta anterior (nos retornos),
  conduta e retorno, com botão "Copiar para prontuário".
- **Encerrar atendimento.** Grava a consulta no histórico do paciente (com prescrição e
  evolução) e volta para a lista de pacientes.

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

## Como o login funciona (e como trocar por um login de verdade)

A versão atual implementa a **Opção A — perfil local**: usuário e senha ficam no
`localStorage` deste navegador (a senha é guardada como hash SHA-256 com *salt*, nunca em
texto puro) e servem para **separar os pacientes de profissionais que dividem o mesmo
computador**. Não há servidor validando nada, os dados **não sincronizam entre dispositivos**
e quem tiver acesso ao navegador alcança o que está armazenado nele. O próprio app avisa isso
na tela de login. Serve para uso pessoal e demonstração — não para guardar dados que exijam
sigilo garantido por sistema.

Todo acesso a dados passa por um único objeto `DB`, com métodos assíncronos:

```
usuarioAtual()                 criarConta(user, senha, nome)     entrar(user, senha)
sair()                         listarPacientes()                 obterPaciente(id)
salvarPaciente(paciente)       excluirPaciente(id)
lerRascunho()                  salvarRascunho(estado)            limparRascunho()
```

Hoje `var DB = DriverLocal;`. Para migrar para a **Opção B — login e banco reais**
(Firebase Authentication + Firestore, Supabase ou equivalente compatível com hospedagem
estática), basta escrever um driver com essa mesma interface e trocar essa linha: nenhuma
tela do wizard precisa mudar. O esqueleto comentado do driver Firebase está no próprio
arquivo, ao lado do driver local.

## Dados

Chaves usadas no `localStorage`, todas com o prefixo `dpoc_clinico`:

| Chave | Conteúdo |
|-------|----------|
| `dpoc_clinico:usuarios` | perfis criados neste navegador (usuário, nome, hash e salt da senha) |
| `dpoc_clinico:sessao` | id do profissional autenticado no momento |
| `dpoc_clinico:<idUsuario>:pacientes` | pacientes e o histórico de atendimentos daquele profissional |
| `dpoc_clinico:<idUsuario>:rascunho` | atendimento em andamento, para retomar após recarregar |

Nada é enviado a servidores.

## Aviso

Ferramenta educacional de apoio à decisão — não substitui o julgamento clínico.
As referências completas, em ABNT, estão no rodapé da própria página.
