# DPOC Clínico

Ferramenta de apoio à decisão para o manejo da DPOC segundo a estratégia GOLD, em
formato *wizard* (uma pergunta ou decisão por tela, com barra de progresso e botão
de voltar).

Página única: **`index.html`** com HTML, CSS e JavaScript embutidos, sem build e sem servidor
próprio. Funciona abrindo o arquivo no navegador; para sincronizar os pacientes entre
dispositivos, dá para conectar a um projeto Firebase (Authentication + Firestore) — ver
*Como o login funciona*.

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
  próprios pacientes. Funciona local (sem configuração) ou com **login e banco reais na nuvem**,
  sincronizando os pacientes entre computador e celular (ver *Como o login funciona*, abaixo).
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

## Como o login funciona

O app tem **dois modos**, e ele escolhe sozinho conforme houver ou não uma configuração de nuvem:

| | Modo local (padrão) | Modo nuvem (Firebase) |
|---|---|---|
| Login | usuário e senha guardados no navegador | e-mail e senha de verdade (Firebase Authentication) |
| Pacientes | só neste navegador | salvos no Firestore, aparecem em qualquer dispositivo |
| Precisa de configuração | não | sim, uma vez (passo a passo abaixo) |
| Custo | nenhum | nenhum, dentro do plano gratuito do Firebase |

Em qualquer um dos modos, o **atendimento em andamento** (o rascunho, antes de "Encerrar
atendimento") fica no navegador do aparelho em que você está digitando — é estado de trabalho,
não vai para a nuvem a cada tecla.

### Modo local

Usuário e senha ficam no `localStorage` deste navegador (a senha como hash SHA-256 com *salt*,
nunca em texto puro) e servem para **separar os pacientes de profissionais que dividem o mesmo
computador**. Não há servidor validando nada, os dados **não sincronizam entre dispositivos** e
quem tiver acesso ao navegador alcança o que está armazenado nele. Serve para uso pessoal e
demonstração — não para dados que exijam sigilo garantido por sistema.

> **Este repositório já está ligado** ao projeto Firebase `dpoc-clinico`: a configuração está
> embutida em `FIREBASE_CONFIG`, no início do `<script>` do `index.html`. Todo dispositivo que
> abrir https://eckermann33.github.io/DPOC-Clinico/ entra no modo nuvem — basta fazer login.
> As instruções abaixo servem para criar outro projeto ou refazer a configuração.

### Ligar a sincronização entre dispositivos (Firebase)

Dá para fazer tudo pela tela **"Ativar sincronização entre dispositivos"**, no rodapé da tela de
login, que traz o mesmo passo a passo. Resumindo:

1. Em **console.firebase.google.com**, entre com sua conta Google e clique em **Criar projeto**.
   Dê um nome (ex.: `dpoc-clinico`); pode desativar o Google Analytics.
2. Menu **Criação → Authentication → Vamos começar**, escolha **E-mail/senha** e ative.
3. Menu **Criação → Firestore Database → Criar banco de dados**, região `southamerica-east1`,
   iniciando em **modo de produção**.
4. Aba **Regras** do Firestore: apague o conteúdo, cole o bloco abaixo e clique em **Publicar**.
   É isto que garante que cada conta leia e escreva apenas os próprios pacientes.

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /usuarios/{uid}/{documento=**} {
         allow read, write: if request.auth != null && request.auth.uid == uid;
       }
     }
   }
   ```

5. **Visão geral do projeto → ícone `</>` (Web)**, registre um app com qualquer apelido e copie o
   bloco `const firebaseConfig = { ... }`.
6. Cole esse bloco **dentro de `var FIREBASE_CONFIG = { }`**, no início do `<script>` do
   `index.html`, e publique. Assim todo dispositivo que abrir o endereço já entra no modo nuvem.
   (Alternativa sem editar o arquivo: colar o bloco na tela "Ativar sincronização" dentro do app —
   mas aí é preciso repetir em cada dispositivo.)

As chaves do `firebaseConfig` são **públicas por definição** no Firebase para web e podem ir para
o repositório: quem protege os dados são o Authentication e as regras do passo 4, não o segredo
da chave.

### Publicar (necessário para o modo nuvem)

Com a sincronização ligada, o site precisa ser aberto por um endereço **https** — abrindo o
arquivo direto do disco (`file://`) o navegador bloqueia a conexão com o Firebase, e o app avisa
isso e volta para o modo local. Publicar pelo GitHub Pages resolve: no repositório, **Settings →
Pages → Source: Deploy from a branch**, escolha a branch e a pasta `/ (root)` e salve. Em poucos
minutos o site fica em `https://<seu-usuario>.github.io/DPOC/`.

### Migrar os pacientes que já estão no computador

Ao entrar no modo nuvem, se houver pacientes salvos em perfis locais deste navegador, a tela de
pacientes mostra um aviso com o botão **"Enviar para a nuvem agora"**. A cópia local não é apagada.

### Trocar de serviço

Todo acesso a dados passa por um objeto `DB` com esta interface:

```
usuarioAtual()                 criarConta(user, senha, nome)     entrar(user, senha)
sair()                         listarPacientes()                 obterPaciente(id)
salvarPaciente(paciente)       excluirPaciente(id)               redefinirSenha(email)
lerRascunho()                  salvarRascunho(estado)            limparRascunho()
```

Existem dois drivers no arquivo — `DriverLocal` e `DriverFirebase` — e o app escolhe qual usar no
início da execução. Para trocar por Supabase ou outro serviço, basta escrever um terceiro driver
com a mesma interface: nenhuma tela do wizard precisa mudar.

## Dados

No modo nuvem, cada paciente é um documento em `usuarios/{uid}/pacientes/{idPaciente}` no
Firestore. No modo local, e para o rascunho do atendimento em qualquer um dos modos, valem estas
chaves do `localStorage`:

| Chave | Conteúdo |
|-------|----------|
| `dpoc_clinico:usuarios` | perfis locais criados neste navegador (usuário, nome, hash e salt da senha) |
| `dpoc_clinico:sessao` | id do perfil local autenticado no momento |
| `dpoc_clinico:<idUsuario>:pacientes` | pacientes e histórico de atendimentos daquele perfil local |
| `dpoc_clinico:<idUsuario>:rascunho` | atendimento em andamento (modo local) |
| `dpoc_clinico:nuvem:<uid>:rascunho` | atendimento em andamento (modo nuvem) |
| `dpoc_clinico:firebase` | configuração do projeto, quando colada pela tela do app |

## Estilo

Direção visual **editorial clínica**: papel off-white quente, títulos em serifada,
interface em Inter, fios finos de 1px no lugar de sombras e cantos quase retos
(2–4 px). Azul-petróleo é a cor institucional; âmbar é o único acento e marca
sempre "onde você está / o que você escolheu" (etapa ativa, barra de progresso,
opção selecionada, checkbox marcado).

| Token | Valor | Uso |
|-------|-------|-----|
| `--paper` | `#faf8f5` | fundo da página |
| `--surface` | `#ffffff` | cards e painéis |
| `--ink` | `#1a1a1a` | texto principal |
| `--rule` / `--rule-strong` | `#e4ded4` / `#cdc5b8` | fios e bordas |
| `--petrol-900` | `#12303f` | botão primário, veredito, marca |
| `--amber` | `#b45309` | acento de estado e seleção |
| `--serif` | Source Serif 4 | títulos, números, veredito |
| `--font` | Inter | corpo, rótulos e controles |

As fontes vêm do Google Fonts; sem internet o *fallback* cai em Georgia + system
sans e o layout continua idêntico. Todo o estilo está no bloco `<style>` do
`index.html`, com as cores em variáveis CSS no `:root` — trocar a paleta inteira
é editar esse bloco.

## Aviso

Ferramenta educacional de apoio à decisão — não substitui o julgamento clínico.
As referências completas, em ABNT, estão no rodapé da própria página.
