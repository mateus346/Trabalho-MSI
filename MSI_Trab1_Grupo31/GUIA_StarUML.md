# StarUML — Lista de elementos por diagrama
**Sistema de Gestão de Tutorias UTC-INF — Fases 1 e 2 (ICONIX)**

Guia para recriar o modelo no StarUML e gerar o ficheiro `.mdj`. Cada secção diz exatamente o que criar.

---

## Como montar no StarUML (rápido)

- **Casos de uso:** `Model` → `Add Diagram` → *Use Case Diagram*. Arrasta **Actor** e **UseCase**. Para os pacotes, cria **Package** e mete os UC lá dentro. O *include* é uma **Dependency** com estereótipo `«include»` (UseCase → UseCase).
- **Classes (modelo de domínio):** *Class Diagram*. Em cada **Class**, duplo-clique para abrir e adicionar **Attributes** e **Operations**. Relações: **Generalization** (triângulo), **Association** (põe a multiplicidade e o nome do papel em cada ponta), **Aggregation** (losango vazio), **Composition** (losango cheio).
- **Robustez:** cria um *Class Diagram* (ou *Communication Diagram*) e adiciona **Class** com o **stereotype** `«boundary»`, `«control»` ou `«entity»` (campo *Stereotype* nas propriedades, ou usa o perfil de análise). Liga os objetos com **Association** e escreve a mensagem no nome da ligação. Regra: ator↔boundary; boundary↔control; entity↔control; control↔boundary/entity/control.

> Dica: os ficheiros `.puml` que já tens têm as **ligações exatas** de cada diagrama de robustez — usa-os lado a lado com esta lista.

---

## 1. Diagrama de Casos de Uso

**Atores (4):** Coordenador · Tutor · Tutorando · **Diretor de Curso** *(proposto)*

**Casos de uso (13):**
| Id | Nome |
|----|------|
| UC01 | Autenticar |
| UC02 | Registar aluno |
| UC03 | Associar aluno a tutor |
| UC04 | Enviar notificação |
| UC05 | Registar-se no sistema |
| UC06 | Preencher dados pessoais e proveniência |
| UC07 | Preencher ficha de frequência |
| UC08 | Solicitar reunião |
| UC09 | Consultar/alterar dados do tutorando |
| UC10 | Convocar reunião |
| UC11 | Registar reunião |
| UC12 | Consultar estatísticas de aproveitamento por UC *(proposto)* |
| UC13 | Consultar relatório de atividade de tutoria *(proposto)* |

**Pacotes:**
- **Acesso e Notificações:** UC01, UC04
- **Gestão de Alunos e Atribuições:** UC02, UC03
- **Dados do Tutorando:** UC05, UC06, UC07, UC09
- **Reuniões:** UC08, UC10, UC11
- **Relatórios:** UC12, UC13

**Associações Ator → Caso de Uso:**
- Coordenador → UC02, UC03
- Tutorando → UC05, UC06, UC07, UC08
- Tutor → UC09, UC10, UC11
- Diretor de Curso → UC12, UC13

**Relações «include»:**
- UC02 → UC03
- UC03 → UC04
- UC07 → UC04
- UC08 → UC04
- UC01 é incluído por: UC02, UC06, UC07, UC08, UC09, UC10, UC11, UC12, UC13
- ⚠️ **UC05 NÃO inclui UC01** (o aluno ainda não tem conta).

---

## 2. Modelo de Domínio / Diagrama de Classes

As **operações** e os atributos marcados com ★ só aparecem no **diagrama atualizado (fim da Fase 2)**. Para o **modelo de domínio de alto nível da Fase 1**, cria as mesmas classes **sem operações** e **sem os atributos ★**.

**Classes e membros:**

| Classe | Atributos | Operações (só Fase 2) |
|--------|-----------|------------------------|
| **Utilizador** | nome, morada, email, telefone, ★dataNascimento, username, password, ★estadoConta | ★autenticar(), ★criarConta() |
| **Coordenador** | numeroFuncionario | — |
| **Tutor** | numeroDocente, gabinete | ★obterTutorandos() |
| **Tutorando** | numeroAluno, anoIngresso, justificacaoCurso, ★dataAtribuicao | ★registar(), ★alterarDados(), ★associarTutor() |
| **Proveniencia** | instituicaoAnterior, areaFormacao | — |
| **Curso** | nome, grau | — |
| **UnidadeCurricular** | nome, ano, semestre, ects | — |
| **FichaFrequencia** | dataPreenchimento | ★guardar() |
| **RegistoUC** | comentarioAulas, ★anoLetivo | — |
| **Avaliacao** | epoca, nota | — |
| **Reuniao** | ★dataPedido, ★assuntoPedido, ★datasSugeridas, data, hora, local, ★estado, assuntosTratados | ★solicitar(), ★convocar(), ★registar() |
| **Mensagem** | data, tipo, texto, ★estadoEnvio | ★enviar() |

**Enumerados (valores possíveis, podes pôr como nota):**
- Reuniao.estado: pendente | convocada | realizada
- Mensagem.estadoEnvio: enviada | pendente
- Utilizador.estadoConta: sem conta | ativa
- Avaliacao.epoca: normal | recurso | especial

**Relações (tipo — de → para — multiplicidades — papel):**
| Tipo | A | B | Mult. A | Mult. B | Nome/papel |
|------|---|---|---------|---------|-----------|
| Generalização | Coordenador | Utilizador | — | — | (herança) |
| Generalização | Tutor | Utilizador | — | — | (herança) |
| Generalização | Tutorando | Utilizador | — | — | (herança) |
| Associação | Tutor | Tutorando | 1 | 0..* | tutoria |
| Associação ★ | Coordenador | Tutorando | 1 | 0..* | atribui |
| **Composição** | Tutorando | Proveniencia | 1 | 1 | — (losango no Tutorando) |
| Associação | Tutorando | Curso | 0..* | 1 | frequenta |
| **Agregação** | Curso | UnidadeCurricular | 1 | 1..* | — (losango vazio no Curso) |
| Associação | Tutorando | FichaFrequencia | 1 | 0..* | preenche |
| **Composição** | FichaFrequencia | RegistoUC | 1 | 1..* | — (losango na Ficha) |
| Associação | RegistoUC | UnidadeCurricular | 0..* | 1 | refere-se a |
| **Composição** | RegistoUC | Avaliacao | 1 | 0..* | — (losango no RegistoUC) |
| Associação | Reuniao | Tutor | 0..* | 1 | convoca/regista |
| Associação ★ | Reuniao | Tutorando | 0..* | 1 | diz respeito a |
| Associação | Reuniao | Utilizador | 0..* | 1..* | presentes |
| Associação | Reuniao | Utilizador | 0..* | 1 | solicitadaPor |
| Associação | Mensagem | Utilizador | 0..* | 1 | destinatário |

---

## 3. Diagramas de Robustez (13)

Para cada UC: cria o(s) **ator(es)**, os objetos `«boundary»`, `«control»`, `«entity»` e liga-os. As ligações exatas (com as mensagens) estão nos `.puml`; abaixo tens o **inventário de objetos** de cada um.

**UC01 – Autenticar**
- Ator: Utilizador
- boundary: formLogin, pagMenuPrincipal, janMensagemErro
- control: verificarCredenciais
- entity: utilizador

**UC02 – Registar aluno**
- Ator: Coordenador
- boundary: formRegistoAluno, janConfirmacao, janMensagemErro
- control: registarAluno
- entity: aluno, curso

**UC03 – Associar aluno a tutor**
- Atores: Coordenador, Tutor, Tutorando
- boundary: formAtribuicao, janNotificacaoTutor, janNotificacaoTutorando
- control: atribuirTutor, enviarNotificacao
- entity: aluno, tutor, mensagem

**UC04 – Enviar notificação**
- Atores: Tutor, Tutorando
- boundary: janNotificacaoTutor, janNotificacaoTutorando
- control: enviarNotificacao
- entity: mensagem, utilizador

**UC05 – Registar-se no sistema**
- Ator: Tutorando
- boundary: formRegisto, janConfirmacao, janMensagemErro
- control: registarConta
- entity: aluno, utilizador

**UC06 – Preencher dados pessoais e proveniência**
- Ator: Tutorando
- boundary: formDadosPessoais, janConfirmacao, janMensagemErro
- control: gravarDadosTutorando
- entity: tutorando, proveniencia

**UC07 – Preencher ficha de frequência**
- Atores: Tutorando, Tutor
- boundary: formFichaFrequencia, janNotificacaoTutor, janMensagemErro
- control: obterUCs, guardarFicha, notificarTutor
- entity: unidadeCurricular, fichaFrequencia, avaliacao, mensagem

**UC08 – Solicitar reunião**
- Atores: Tutorando, Tutor
- boundary: formPedidoReuniao, janNotificacaoTutor, janMensagemErro
- control: registarPedido, enviarNotificacao
- entity: reuniao, tutorando, mensagem

**UC09 – Consultar/alterar dados do tutorando**
- Ator: Tutor
- boundary: pagListaTutorandos, pagDadosTutorando, formAlteracaoDados, janMensagemErro
- control: obterDadosTutorando, verificarDados
- entity: tutorando, proveniencia, fichaFrequencia

**UC10 – Convocar reunião**
- Ator: Tutor
- boundary: formConvocarReuniao, janConfirmacao, janMensagemErro
- control: convocarReuniao
- entity: reuniao, tutorando

**UC11 – Registar reunião**
- Ator: Tutor
- boundary: formRegistoReuniao, janConfirmacao, janMensagemErro
- control: registarReuniao
- entity: reuniao, utilizador

**UC12 – Consultar estatísticas de aproveitamento por UC** *(proposto)*
- Ator: Diretor de Curso
- boundary: formFiltrosEstatisticas, pagEstatisticas, janMensagemErro
- control: calcularEstatisticas
- entity: unidadeCurricular, registoUC, avaliacao

**UC13 – Consultar relatório de atividade de tutoria** *(proposto)*
- Ator: Diretor de Curso
- boundary: formFiltrosRelatorio, pagRelatorioTutoria, janMensagemErro
- control: gerarRelatorio
- entity: tutor, tutorando, reuniao

---

### Ordem sugerida de trabalho no StarUML
1. Diagrama de **casos de uso** (atores + UC + pacotes + include).
2. Diagrama de **classes de domínio (Fase 1)** — sem operações nem atributos ★.
3. Os **13 diagramas de robustez** (um por UC) — copia as ligações dos `.puml`.
4. **Atualiza o diagrama de classes (Fase 2)** — acrescenta as operações e os atributos ★ descobertos na robustez.
5. `File` → `Save` gera o **`.mdj`** para entregar.
