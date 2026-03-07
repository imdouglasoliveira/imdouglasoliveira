# Guia de Atualização do Profile README

Como regenerar os SVGs animados do perfil GitHub e publicar as alterações.

---

## Pré-requisitos

- Python 3.9+
- Dependências instaladas:
  ```bash
  cd packages/imdouglasoliveira
  pip install -r requirements.txt
  ```

---

## Atualização Rápida (só regenerar SVGs)

```bash
cd packages/imdouglasoliveira
python -m generator.main
```

Isso busca stats atualizados da API do GitHub e regenera os 4 SVGs em `assets/generated/`.

> **Nota:** Sem `GITHUB_TOKEN`, commits privados não são contados (usa API REST pública como fallback). O GitHub Action em produção fornece o token automaticamente.

---

## Alterar Conteúdo do Perfil

Edite `config.yml` e regenere:

```bash
# 1. Editar o config
code config.yml    # ou qualquer editor

# 2. Regenerar
python -m generator.main

# 3. Conferir resultado (abrir SVGs no browser)
start assets/generated/galaxy-header.svg
```

### O que dá pra mudar no `config.yml`

| Campo | O que faz |
|-------|-----------|
| `profile.name` / `tagline` | Nome e subtítulo no header |
| `profile.bio` | Texto "About me" |
| `profile.philosophy` | Citação exibida no galaxy header |
| `galaxy_arms` | Até 3 braços com tecnologias (cores: `synapse_cyan`, `dendrite_violet`, `axon_amber`) |
| `projects` | Até 3 projetos em destaque (repo, arm, description) |
| `social` | Links: email, linkedin, website |
| `theme` | Override de cores (9 hex values) |
| `languages.exclude` | Linguagens a esconder do gráfico |
| `stats.metrics` | Métricas exibidas: commits, stars, prs, issues, repos |

---

## Modo Demo (sem API)

Para testar mudanças sem consumir rate limit da API:

```bash
python -m generator.main --demo
```

Usa dados fictícios de stats e linguagens.

---

## Setup Interativo (criar config do zero)

```bash
python -m generator.main init
```

Wizard que pergunta nome, tagline, tecnologias, etc. e gera o `config.yml`.

---

## Publicar no GitHub

Após regenerar os SVGs:

```bash
cd packages/imdouglasoliveira

# Verificar alterações
git status
git diff --stat

# Commit e push
git add assets/generated/ config.yml
git commit -m "chore: regenerate profile SVGs"
git push origin main
```

> Remote configurado: `git@github-imdouglas:imdouglasoliveira/imdouglasoliveira.git`

---

## Regeneração Automática

O GitHub Action (`.github/workflows/generate-profile.yml`) regenera automaticamente:

- **A cada 12 horas** (cron)
- **Ao alterar** `config.yml` ou `generator/**` (push)
- **Manual** via GitHub Actions → "Run workflow"

O Action usa `GITHUB_TOKEN` (fornecido pelo GitHub) para stats completos e faz auto-commit dos SVGs.

---

## Rodar Testes

```bash
pip install -r requirements-dev.txt
pytest                              # todos
pytest tests/test_config.py -v      # validação de config
pytest tests/test_svg_generation.py  # geração de SVGs
```

---

## Arquivos Gerados

| SVG | Dimensão | Conteúdo |
|-----|----------|----------|
| `galaxy-header.svg` | 850×280 | Galáxia animada com nome, tagline e tecnologias |
| `stats-card.svg` | 850×180 | Métricas GitHub (commits, stars, PRs, issues, repos) |
| `tech-stack.svg` | 850×auto | Barras de linguagens + radar de setores |
| `projects-constellation.svg` | 850×220 | Cards de projetos em destaque |

---

## Troubleshooting

| Problema | Causa | Solução |
|----------|-------|---------|
| `commits: 0` | Sem token, REST não conta privados | Normal localmente. Action usa token. |
| `rate limit low` | Muitas chamadas à API | Esperar reset ou usar `--demo` |
| `ModuleNotFoundError` | Deps não instaladas | `pip install -r requirements.txt` |
| SVGs não mudam | Config não alterado | Editar `config.yml` e regenerar |
