# docs-petacademy

## Passo 3: Instalar e Configurar Next.js

### 3.1 Criar um Novo Projeto Next.js
No diretório do repositório, execute:
```bash
npx create-next-app@latest .
```

### 3.2 Instalar Nextra
Instale o Nextra para a documentação:
```bash
npm install nextra nextra-theme-docs
```

### 3.3 Configurar Nextra
Crie ou edite o arquivo `next.config.js` para usar o Nextra:
```javascript
// next.config.js
const withNextra = require('nextra')('./theme.config.js');
module.exports = withNextra();
```

Em seguida, crie um arquivo `theme.config.js` na raiz do projeto:
```javascript
// theme.config.js
export default {
  projectLink: 'https://github.com/seuusuario/documentation-repo',
  docsRepositoryBase: 'https://github.com/seuusuario/documentation-repo/blob/main/',
  titleSuffix: ' – Sua Documentação',
  logo: () => (
    <>
      <span>Documentação</span>
    </>
  ),
};
```

## Passo 4: Criar Script de Automação

### 4.1 Instalar Dependências Necessárias
Instale a biblioteca `@octokit/rest` para interagir com a API do GitHub:
```bash
npm install @octokit/rest
```

### 4.2 Criar Script para Gerar Markdown das Issues
Crie um arquivo chamado `generateDocs.js` na raiz do projeto com o seguinte conteúdo:
```javascript
const { Octokit } = require("@octokit/rest");
const fs = require("fs");
const path = require("path");

const octokit = new Octokit({ auth: 'SEU_GITHUB_TOKEN' });

async function generateDocsFromIssues(owner, repo, milestoneId) {
    const { data: issues } = await octokit.issues.listForRepo({
        owner,
        repo,
        milestone: milestoneId,
        state: 'all',
    });

    issues.forEach(issue => {
        const filePath = path.join(__dirname, 'docs', `${issue.number}.md`);
        const content = `# ${issue.title}\n\n${issue.body}`;
        fs.writeFileSync(filePath, content);
    });
}

generateDocsFromIssues('seuusuario', 'documentation-repo', 'ID_DA_MILESTONE');
```
Substitua `SEU_GITHUB_TOKEN`, `seuusuario`, `documentation-repo`, e `ID_DA_MILESTONE` pelos valores apropriados.

## Passo 5: Configurar GitHub Actions para Automatização

Crie um arquivo de workflow em `.github/workflows/generate-docs.yml` com o seguinte conteúdo:
```yaml
name: Generate Docs

on:
  push:
    branches:
      - main

jobs:
  generate-docs:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      
      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '14'

      - name: Install dependencies
        run: npm install

      - name: Generate documentation from GitHub issues
        run: node generateDocs.js
```

## Passo 6: Testar a Configuração

1. Adicione algumas issues à sua milestone no GitHub.
2. Faça um commit e push das suas alterações:
```bash
git add .
git commit -m "Configuração inicial do projeto"
git push origin main
```

Após fazer isso, o GitHub Actions deve ser acionado automaticamente e executar o script que gera os arquivos Markdown baseados nas issues da sua milestone.
