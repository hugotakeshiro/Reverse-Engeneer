# REA no dia a dia — guia rápido em português

Este repositório é o **REA (Reverse Engineer Anything)**: um servidor MCP + linha de
comando que dá ao Claude (Claude Desktop ou Claude Code) ferramentas para **olhar por
dentro de outros aplicativos** — sites, apps web/JavaScript, apps de desktop (Electron),
APKs Android e programas compilados — e explicar **como uma funcionalidade foi feita**,
com evidências, para você aplicar a ideia no seu próprio produto.

> Tudo roda **localmente no seu computador**. Nada do app analisado é enviado para
> servidores do REA.

---

## 1. Status desta cópia (verificado em 02/11/2026)

| Verificação                                               | Resultado                             |
| --------------------------------------------------------- | ------------------------------------- |
| `npm ci` + `npm run build:cached`                         | ✅ compilou sem erros                 |
| `npm run check:fast` (tipos + lint)                       | ✅ passou                             |
| Testes do analisador JavaScript (11 arquivos, 273 testes) | ✅ todos passaram                     |
| Servidor MCP (`rea --mcp`)                                | ✅ inicia e expõe **206 ferramentas** |
| Análise real de um app (Excalidraw 0.18, 65 arquivos JS)  | ✅ concluída em 1min17s               |

Nenhum código do repositório foi alterado — este guia é o único arquivo adicionado.

> ℹ️ Esta cópia (fork) está na versão **6.4.0**; a versão oficial mais recente no npm é
> **6.8.0**. Para usar no dia a dia, o caminho recomendado (seção 2) já baixa sempre a
> versão mais nova. Se quiser atualizar o fork também, use o botão **“Sync fork”** na
> página do repositório no GitHub.

---

## 2. Instalar no seu computador (uma vez só, ~5 minutos)

### Pré-requisito: Node.js

Instale o **Node.js 22 LTS ou 24 LTS** em <https://nodejs.org> (botão “LTS”).
Versões 23 e 25 **não** são suportadas.

### Passo a passo

1. **Feche totalmente o Claude Desktop** (no Windows: ícone perto do relógio → _Quit_).
   Se ele estiver aberto, pode sobrescrever a configuração.
2. Abra o **Terminal** (Mac) ou **PowerShell** (Windows) e rode:

   ```bash
   npx rea-agents@latest setup
   ```

3. Quando perguntar quais agentes configurar, marque **Claude Desktop** (e **Claude Code**,
   se usar). Revise as mudanças propostas e aprove. O setup faz **backup** das suas
   configurações antes de mexer.
4. Se perguntar sobre instalar o **Hopper** (só serve para apps compilados de Mac/Linux),
   pode responder **não** por enquanto.
5. Abra o Claude Desktop de novo.
6. Para conferir se está tudo ok:

   ```bash
   npx rea-agents@latest doctor
   ```

   Avisos sobre _Hopper / Ghidra / IDA ausentes_ são normais — eles só são necessários
   para programas compilados (.exe, apps nativos). Para sites e apps JavaScript, não precisa.

---

## 3. Como pedir ao Claude (prompts prontos)

Depois de instalado, é só conversar no Claude Desktop. Modelo geral:

```text
Use o REA para entender como funciona [FUNCIONALIDADE] no [APP/SITE],
me mostre as evidências e depois me diga como construir algo parecido
no meu projeto [NOME DO PROJETO].
```

### Exemplos pensados para o seu trabalho

**App de registro de treino (para o openGym)**

```text
Use o REA para analisar o app web [nome/link] que eu abri no navegador.
Quero entender como funciona o cronômetro de descanso entre séries e como
ele salva o histórico de treino (localStorage? servidor?). Depois proponha
como implementar isso no meu openGym.
```

**Funil de agendamento de um concorrente (para a SuperaDOR / CRM)**

```text
Use o REA para inspecionar a página de agendamento de [site]. Quero saber
quais etapas o formulário tem, para quais serviços ele envia os dados
(ex.: WhatsApp, Calendly, CRM) e como captura o lead. Depois desenhe um
fluxo equivalente para captar leads da consultoria online SuperaDOR e do
atendimento presencial em Paulínia.
```

**App de desktop feito em Electron (ex.: um editor de vídeo ou de notas)**

```text
Use o REA para analisar o app instalado em [caminho da pasta do app].
Explique como funciona a busca/exportação/[recurso] e quais bibliotecas
ele usa. Não copie código — quero só entender a lógica para fazer a minha versão.
```

**App Android (APK)**

```text
Use o REA para inspecionar o APK em [caminho]. Liste as telas, permissões
e como funciona [recurso]. (Se faltar jadx/apktool, me diga o que instalar.)
```

### O que esperar da resposta

O REA separa sempre:

- **Observado** — está escrito no código do app (com arquivo e posição exata);
- **Inferido** — conclusão do analista a partir das observações;
- **Desconhecido** — o que a análise não conseguiu ver (isso **não** significa que não existe).

Peça ao Claude para respeitar essa separação ao transformar a análise em plano.

---

## 4. Do “como eles fazem” ao “meu produto” — fluxo recomendado

1. **Escolha uma funcionalidade só** por vez (ex.: “cronômetro de descanso”, não “o app todo”).
2. **Investigue com o REA** → peça as evidências.
3. **Peça uma especificação** em linguagem simples: telas, dados salvos, regras, integrações.
4. **Construa a sua versão** a partir da especificação (no Lovable, no openGym, no CRM…),
   com seu próprio código, textos e visual.
5. **Diferencie**: adicione o que só você tem (base científica, avaliação de força,
   acompanhamento fisioterapêutico).

---

## 5. Usar pelo terminal (opcional)

```bash
# Instalar o comando "rea" de forma permanente
npm install --global rea-agents
rea --help

# Visão geral de um app JavaScript/Electron (pasta ou arquivo .asar)
rea analyze-javascript-application "C:/caminho/do/app" --json > analise.json

# Rastrear uma funcionalidade a partir da análise salva (ver docs/javascript-application-workflows.md)
rea trace-application-feature ./trace-input.json --json

# Atualizar / remover
rea update
rea uninstall
```

### Rodando direto desta cópia do repositório (para desenvolvedores)

```bash
git clone https://github.com/hugotakeshiro/reverse-engeneer
cd reverse-engeneer
npm ci
npm run build:cached
node scripts/rea.mjs --help
```

---

## 6. Demonstração feita na implementação

Para validar, o REA analisou o pacote de produção do **Excalidraw 0.18** (app web de
desenho, código aberto, licença MIT):

- 65 arquivos JavaScript lidos (4,4 MB), 340.960 nós de sintaxe visitados, 0 falhas;
- 2.405 módulos/exports mapeados no grafo do aplicativo;
- encontrou como o app **guarda dados** (4 pontos de `localStorage`), que ele aceita
  **arquivos compartilhados** pelo celular (_Web Share Target_) e usa um **Web Worker**;
- rastreou a única chamada de rede (`fetch`) até a função que **baixa um módulo WebAssembly**
  — ou seja, o app desenha offline e só busca esse componente.

Observação honesta: em apps muito grandes e minificados, o rastreamento fino entre
funções fica parcial (o REA avisa quando isso acontece). Funciona melhor quando você pede
**uma funcionalidade específica** e deixa o Claude combinar a análise com a leitura do código.

---

## 7. Limites de uso (importante)

O REA é para **entender e se inspirar**. Pelo próprio [DISCLAIMER](DISCLAIMER.md) do projeto, **não** use para:

- copiar código, imagens, textos ou marcas de outro app;
- burlar login, pagamento, licença, DRM ou proteções;
- acessar sistemas/contas sem autorização.

Analise apps que você instalou ou páginas públicas, e construa a sua versão do zero.

---

## 8. Documentação completa (em inglês)

- [README principal](README.md) · [Instalação](docs/installation.md) · [CLI](docs/cli.md)
- [Apps JavaScript/Electron](docs/javascript-application-workflows.md)
- [Observação no navegador](docs/browser-observation.md) · [Android](docs/android-analysis.md)
- README em português do Brasil: [docs/readme/README_pt-BR.md](docs/readme/README_pt-BR.md)
