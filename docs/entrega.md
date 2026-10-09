# Relatório de Entrega — Prática de GitHub Secrets

## 1. Explicação do Problema Encontrado

No projeto original, a informação confidencial (`MENSAGEM_SECRETA`) estava definida diretamente em texto puro (*hardcoded*) dentro do arquivo de pipeline `.github/workflows/main.yml`:

```yaml
env:
  MENSAGEM_SECRETA: "Minha mensagem secreta"
```

### Problemas identificados:
- **Exposição de dados sensíveis**: Como o arquivo `.github/workflows/main.yml` faz parte do repositório Git, qualquer pessoa com acesso de leitura ao repositório (colaboradores ou qualquer usuário da internet, caso o repositório seja público) tem acesso direto ao valor do segredo.
- **Histórico versionado permanente**: Mesmo que o valor fosse apagado em commits futuros, ele permaneceria registrado no histórico do Git (`git log`), expondo a credencial.
- **Vulnerabilidade crítica de segurança**: Se essa variável representasse uma chave de API, token de acesso, credencial de nuvem ou senha de banco de dados, o sistema ficaria vulnerável a invasões, vazamento de dados, uso indevido de recursos e sequestro de contas.

---

## 2. Configuração do GitHub Secret

- **Nome do Secret utilizado:** `MENSAGEM_SECRETA`
- **Onde foi configurado:** No painel web do GitHub, em:
  `Settings` → `Secrets and variables` → `Actions` → `Repository secrets`
- **Valor cadastrado:** Definido diretamente na interface segura do GitHub (exemplo: `Esta mensagem está protegida pelo GitHub Secrets`). Esse valor fica criptografado em repouso e nunca é exibido na interface após salvo.

---

## 3. Explicação do Fluxo: Do Secret até a Aplicação

O ciclo de vida da informação confidencial segue o fluxo:

$$\text{GitHub Secret} \longrightarrow \text{GitHub Actions Runner} \longrightarrow \text{Variável de Ambiente (OS)} \longrightarrow \text{Aplicação Node.js}$$

1. **GitHub Secret**: O segredo fica armazenado de forma segura e criptografada pela infraestrutura do GitHub.
2. **Injeção no Workflow**: No arquivo `.github/workflows/main.yml`, o GitHub Actions recupera o valor utilizando a sintaxe de interpolação:
   ```yaml
   env:
     MENSAGEM_SECRETA: ${{ secrets.MENSAGEM_SECRETA }}
   ```
3. **Variável de Ambiente**: Durante a execução do job no container/máquina virtual do runner Ubuntu, o GitHub Actions injeta esse valor no ambiente do processo do sistema operacional como uma variável de ambiente chamada `MENSAGEM_SECRETA`. Além disso, o GitHub Actions aplica uma máscara automática (*masking*), substituindo qualquer impressão acidental desse valor nos logs de execução por `***`.
4. **Aplicação Node.js**: O script Node.js (`server.js`) consome a variável através do objeto global `process.env`:
   ```javascript
   const mensagem = process.env.MENSAGEM_SECRETA;
   ```
   Dessa forma, o código-fonte da aplicação e os arquivos do repositório não precisam conter o valor real do segredo.

---

## 4. Respostas das Questões Propostas

### Questões da Seção 3 (Situação Inicial)

1. **Onde o segredo está armazenado?**
   - Na situação inicial, o segredo estava armazenado diretamente em texto claro dentro do arquivo `.github/workflows/main.yml`, versionado no repositório Git.

2. **Quem poderia visualizar essa informação?**
   - Qualquer pessoa com acesso de visualização ao repositório Git (membros da equipe, colaboradores e, se o repositório for público, qualquer pessoa na internet).

3. **Por que essa prática não é recomendada?**
   - Porque viola princípios básicos de segurança da informação (como o princípio do menor privilégio e separação de credenciais e código). Credenciais no código podem ser facilmente copiadas, vazadas acidentalmente ou comprometidas.

4. **O que poderia acontecer se essa informação fosse uma senha ou um token real?**
   - Um invasor poderia autenticar-se em serviços críticos (banco de dados, provedor de nuvem como AWS/GCP/Azure, APIs de pagamento), roubar ou excluir dados de usuários, provisionar recursos indevidos gerando prejuízos financeiros e comprometer toda a infraestrutura da organização.

---

### Questões da Seção 7 (Conceitos e Fixação)

1. **Qual é a diferença entre armazenar o valor diretamente no `pipeline.yml` e armazená-lo no GitHub Secrets?**
   - Armazenar diretamente no arquivo do workflow expõe o valor para qualquer um que leia o repositório e grava o segredo de forma permanente no histórico Git. No GitHub Secrets, o valor fica armazenado de forma criptografada nos servidores do GitHub, fora do código-fonte, não é acessível visualmente após criado e é mascarado automaticamente nos logs de execução.

2. **Onde o valor do Secret fica configurado?**
   - Fica configurado nas configurações de segurança do repositório no GitHub (ou a nível de organização/ambiente): `Settings` → `Secrets and variables` → `Actions`.

3. **O valor do Secret deve ser colocado no código da aplicação?**
   - Não. O código da aplicação deve apenas ler variáveis de ambiente (como `process.env.NOME_VARIAVEL`), mantendo o código desacoplado e seguro contra vazamentos.

4. **Qual é a função de `process.env.MENSAGEM_SECRETA`?**
   - É a propriedade do runtime Node.js que acessa o dicionário de variáveis de ambiente do sistema operacional fornecidas para o processo atual, recuperando o valor atribuído à variável `MENSAGEM_SECRETA`.

5. **Por que utilizar Secrets é importante em uma pipeline de CI/CD?**
   - Pipelines frequentemente precisam de credenciais para realizar testes, compilação e deploy (chaves SSH, tokens de API, credenciais de nuvem, senhas de banco). O gerenciamento via Secrets garante que a automação execute com privilégios necessários sem comprometer a confidencialidade das credenciais.

6. **Se fosse necessário utilizar uma senha real de banco de dados, por que seria inadequado colocar essa senha diretamente no arquivo `.yml`?**
   - Seria inadequado porque qualquer pessoa com acesso ao repositório ou ao histórico de commits teria a senha de acesso direto ao banco de dados, podendo consultar dados sigilosos, alterar registros ou derrubar o banco de produção.

---

## 5. Evidência de Execução da Pipeline

*(Após configurar o Secret e fazer o push para o GitHub, anexe aqui o print ou log da execução da aba Actions comprovando o sucesso do job e a mensagem `Secret recebido pela aplicação com sucesso.`)*
