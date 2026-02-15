# Documento de Visão – Sistema de Gestão de Ocorrências (SGO)

---

## 1. Visão Geral

O modelo de classes foi projetado para representar as principais entidades e seus relacionamentos no processo de gestão de ocorrências disciplinares e medidas educativas no ambiente institucional. A estrutura utiliza herança (generalização) para agrupar os diferentes papéis de usuários a partir da superclasse `Usuario`, centralizando a autenticação (via método `autenticar()`) e dados cadastrais, promovendo reutilização de código e segurança.

As associações entre as classes refletem o fluxo completo:
1. O registro da ocorrência pelo **Professor**;
2. A anexação de provas (**Anexos**);
3. O debate via **Comentários**;
4. A análise pela **Coordenação**;
5. A decisão final (**Medida**) que pode escalar até o **Diretor**.

O modelo garante rastreabilidade através de IDs únicos (UUID) e logs de data/hora.

---

## 2. Diagrama de Classes

O diagrama de classes apresenta uma visão estrutural do sistema, evidenciando:

* **Herança:** A superclasse `Usuario` e suas especializações (`Aluno`, `Professor`, `Coordenacao` e `Diretor`).
* **Agregação:** A classe `Ocorrencia` como entidade central, agregando `Anexos` (evidências), `Comentarios` (histórico) e gerando `Medidas` (sanções).
* **Comunicação:** A classe `Notificacao`, responsável pela comunicação assíncrona com os envolvidos.
* **Comportamento:** Métodos de ação explícitos, como `registrarOcorrencia()`, `aplicarMedida()` e `baixar()` (geração de documentos).

<img width="2528" height="1696" alt="Image" src="https://github.com/user-attachments/assets/0f254a90-5e22-4b85-8200-9149b9974a75" />

---

## 3. Catálogo de Classes

| Classe | Descrição | Atributos Principais |
| :--- | :--- | :--- |
| **Usuario** | Superclasse que representa qualquer pessoa no sistema. | `idUsuario`, `nome`, `email`, `cpf`, `papel`, `ativo` |
| **Aluno** | Subclasse de Usuario. O estudante sujeito às normas. | `matricula`, `curso`, `turma`, `dataNascimento`, `responsavelId` |
| **Professor** | Subclasse de Usuario. Docente que reporta os fatos. | `siape`, `disciplinas` |
| **Coordenacao** | Subclasse de Usuario. Equipe pedagógica gestora. | `setor`, `permissaoAvaliar` |
| **Diretor** | Subclasse de Usuario. Gestão superior para casos graves. | `nivelDecisao` |
| **Ocorrencia** | O evento disciplinar registrado. | `idOcorrencia`, `tipo`, `descricao`, `status`, `dataHoraRegistro` |
| **Medida** | A sanção ou ação educativa aplicada. | `idMedida`, `tipo`, `duracaoDias`, `status`, `aplicadoPorId` |
| **Anexo** | Arquivos de prova vinculados à ocorrência. | `idAnexo`, `nomeArquivo`, `mimeType`, `caminhoArmazenamento` |
| **Comentario** | Histórico de discussão sobre o caso. | `idComentario`, `texto`, `visibilidade`, `autorId` |
| **Notificacao** | Alertas enviados aos usuários. | `idNotificacao`, `mensagem`, `canal`, `lida` |

---

## 4. Dicionário de Dados

### `Usuario`
* **idUsuario** (UUID): Identificador único. *(Chave Primária)*
* **nome** (String): Nome completo. *(Obrigatório)*
* **email** (String): Email para login. *(Único)*
* **cpf** (String): Cadastro de pessoa física. *(Obrigatório)*
* `autenticar()`: Método que valida as credenciais.

### `Aluno`
* **matricula** (String): Matrícula acadêmica. *(Único)*
* **curso** (String): Curso vinculado.
* **turma** (String): Turma vinculada.
* **responsavelId** (UUID): Referência ao responsável legal.

### `Professor`
* **siape** (String): Matrícula do servidor federal/estadual.
* **disciplinas** (List): Lista de matérias lecionadas.
* `solicitarOcorrencia()`: Método para iniciar um registro.

### `Coordenacao`
* **setor** (String): Área de atuação (ex: Pedagógico, Disciplinar).
* **permissaoAvaliar** (Boolean): Define se pode aplicar medidas.
* `gerarRelatorio()`: Compila dados estatísticos.

### `Ocorrencia`
* **idOcorrencia** (UUID): Identificador da ocorrência.
* **tipo** (Enum): Classificação (Indisciplina, Atraso, Dano material).
* **descricao** (Text): Detalhes do ocorrido.
* **status** (Enum): Aberta, Em Análise, Concluída.
* `baixar()`: Gera PDF da ocorrência.

### `Medida`
* **idMedida** (UUID): Identificador da sanção.
* **tipo** (Enum): Advertência verbal, escrita, suspensão.
* **duracaoDias** (Int): Tempo de afastamento (se houver).
* **dataAplicacao** (Date): Data de vigência.
* `aplicarOcorrenciaId()`: Vincula a medida à ocorrência original.

### `Anexo`
* **idAnexo** (UUID): Identificador do arquivo.
* **nomeArquivo** (String): Nome original.
* **caminhoArmazenamento** (String): URL/Path do arquivo.
* **mimeType** (String): Tipo do arquivo (pdf, jpg).

### `Notificacao`
* **idNotificacao** (UUID): ID do alerta.
* **mensagem** (String): Texto do aviso.
* **canal** (String): Email, Push, SMS.
* **lida** (Boolean): Status de leitura.

---

## 5. Recursos do Produto (Funcionalidades)

* **Autenticação Centralizada:** Login único para todos os perfis via classe `Usuario`.
* **Registro de Ocorrências:** Professores podem registrar incidentes com descrição detalhada.
* **Gestão de Evidências:** Upload de anexos (fotos/docs) vinculados à ocorrência.
* **Fluxo de Discussão:** Sistema de comentários (públicos ou internos) na ocorrência.
* **Aplicação de Medidas:** Coordenação/Direção aplica sanções (advertências/suspensões).
* **Geração de Documentos:** Método `baixar()` permite exportar a ocorrência ou medida em PDF.
* **Notificações Multicanal:** Envio automático de alertas sobre mudanças de status.
* **Hierarquia de Decisão:** Diferenciação entre o que a Coordenação resolve e o que escala para o Diretor.

---

## 6. Casos de Uso

### 6.1 Atores e Diagrama Conceitual
* **Ator Primário:** Professor (Registra), Coordenador (Analisa).
* **Ator Secundário:** Aluno (Consulta), Diretor (Decide casos graves).
* **Sistema:** Valida regras, armazena dados e notifica.

<img width="1024" height="559" alt="Image" src="https://github.com/user-attachments/assets/e20bb7c6-7f2c-4230-acad-cd7eae686ade" />

### 6.2 Fluxos de Caso de Uso

#### Cenário A: Professor Registra Ocorrência
1. O Professor autentica-se no sistema.
2. Seleciona a opção `solicitarOcorrencia()`.
3. O sistema pede: Turma, Aluno, Tipo e Descrição.
4. O Professor insere os dados e, opcionalmente, faz upload de um **Anexo**.
5. O sistema salva a **Ocorrencia** com status "Aberta".
6. A **Coordenacao** é notificada via **Notificacao**.

#### Cenário B: Coordenação Aplica Medida
1. A **Coordenacao** acessa a lista de ocorrências pendentes.
2. Analisa a descrição e os anexos.
3. Pode usar **Comentario** para pedir mais detalhes ao professor.
4. Decide pela aplicação de uma sanção.
5. O sistema altera o status da Ocorrência para "Concluída".
6. O sistema gera uma **Notificacao** para o Aluno e responsável.

#### Cenário C: Diretor Intervém
1. Para casos gravíssimos, a Coordenação escala para o Diretor.
2. O Diretor registra a decisão final na classe **Medida**.

---

## 7. Princípios SOLID Aplicados

O diagrama e a arquitetura foram pensados respeitando os princípios:
- [x] **S** - Single Responsibility Principle
- [x] **O** - Open/Closed Principle
- [x] **L** - Liskov Substitution Principle
- [x] **I** - Interface Segregation Principle
- [x] **D** - Dependency Inversion Principle

---

## 8. Componentes (Arquitetura)

* Camada de Apresentação (Frontend)
* Camada de Aplicação (Backend API)
* Camada de Persistência (Banco de Dados)
* Serviço de Armazenamento (Storage)
* Serviço de Notificação

---

## 9. Restrições

* **9.1 Restrições de desenvolvimento:** (A definir)
* **9.2 Restrições de segurança:** (A definir)
* **9.3 Restrições de metodologia:** (A definir)
