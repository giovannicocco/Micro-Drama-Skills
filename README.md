# Micro-Drama-Skills 🎬

Sistema de produção automatizada de micro-dramas de ponta a ponta, impulsionado por IA. Usa Claude Skills para implementar um workflow completo, desde a escrita do roteiro e design de personagens até geração de storyboard e envio de vídeos.

## Visão geral do projeto

Este projeto fornece um conjunto de **Claude Skills** para gerar micro-dramas automaticamente. Cada obra contém 25 episódios (30 segundos por episódio). O sistema gera automaticamente o roteiro, a definição dos personagens, storyboards em grade 6-frame, configurações de storyboard e também pode chamar APIs de IA para gerar imagens/vídeos, enviando tudo ao final para a pipeline de geração de vídeo do Seedance.

### Capacidades principais

| Skill | Função | Exemplo de comando de disparo |
|------|------|-------------|
| **produce-anime** | Gera um micro-drama completo (roteiro + personagens + storyboard) | "Produza um micro-drama de ficção científica" |
| **generate-media** | Chama a API do Gemini para gerar imagens de personagens / storyboards / vídeos | "Gere as imagens de DM-002" |
| **submit-anime-project** | Envia tarefas em lote para a geração de vídeo do Seedance | "Enviar DM-002 para o Seedance" |

## Estrutura de diretórios

```
.
├── .claude/skills/                   # Definições das Claude Skills
│   ├── produce-anime/SKILL.md        # Skill de produção de micro-drama
│   ├── generate-media/SKILL.md       # Skill de geração de mídia
│   └── submit-anime-project/SKILL.md # Skill de envio de tarefas
├── .config/
│   ├── api_keys.sample.json          # Exemplo de configuração de API
│   ├── api_keys.json                 # Configuração de API (criar manualmente; está no gitignore)
│   └── visual_styles.json            # Presets de estilo visual (10 opções)
├── projects/
│   ├── index.json                    # Índice global de obras
│   ├── DM-001_dhgt/                  # "Luzes no Caminho de Volta"
│   └── DM-002_tjkc/                  # "Fúria do Ouro de Carbono"
└── README.md
```

### Estrutura de diretório de uma obra

```
DM-002_tjkc/
├── metadata.json                     # Metadados da obra
├── script/full_script.md             # Roteiro completo (25 episódios)
├── characters/
│   ├── character_bible.md            # Bíblia de personagens
│   ├── ref_index.json                # Índice de imagens de referência dos personagens
│   ├── 林策_ref.png                   # Imagem de referência do personagem (gitignore)
│   └── ...
├── episodes/
│   ├── EP01/
│   │   ├── dialogue.md               # Roteiro de diálogos
│   │   ├── storyboard_config.json    # Configuração do storyboard (grade 6-frame × duas metades)
│   │   ├── seedance_tasks.json       # Tarefas de envio ao Seedance
│   │   ├── DM-002-EP01-A_storyboard.png  # Storyboard da metade superior (gitignore)
│   │   └── DM-002-EP01-B_storyboard.png  # Storyboard da metade inferior (gitignore)
│   └── ... (EP01-EP25)
├── seedance_project_tasks.json       # Consolidação das tarefas da obra inteira (50 entradas)
├── video_index.json                  # Índice de numeração dos vídeos
└── generate_media.py                 # Script de geração de mídia
```

## Início rápido

### 1. Configurar a API

Copie o arquivo de exemplo e preencha com sua API Key:

```bash
cp .config/api_keys.sample.json .config/api_keys.json
```

Edite `.config/api_keys.json`:

```json
{
  "gemini_api_key": "YOUR_GEMINI_API_KEY",
  "base_url": "https://generativelanguage.googleapis.com/",
  "gemini_image_model": "gemini-2.5-flash-image-preview"
}
```

### 2. Instalar dependências

```bash
python -m venv .venv
source .venv/bin/activate
pip install google-genai Pillow requests
```

### 3. Usar as Claude Skills

Abra este projeto em uma ferramenta compatível com Claude Skills (como Claude Code, OpenClaw etc.). As skills serão carregadas automaticamente.

**Produzir micro-drama:**
```
> Produza um micro-drama com estética retrô de Hong Kong
> Produza um micro-drama escolar em estilo cyberpunk
```

**Gerar mídia:**
```
> Gere as imagens de storyboard de DM-002
> Gere as imagens do episódio 1 ao episódio 5
```

**Enviar tarefas:**
```
> Enviar DM-002 para o Seedance (modo simulação)
```

## Presets de estilo visual

O sistema vem com 10 estilos visuais cinematográficos embutidos. Durante a produção, eles podem ser especificados por nome, ID ou nome em chinês.

| ID | Nome em inglês | Nome em chinês | Câmera / características |
|----|--------|--------|------------|
| 1 | Cinematic Film | 电影质感 | Panavision Sphero 65, Vision3 500T (**padrão**) |
| 2 | Anime Classic | 经典动漫 | estilo desenhado à mão do Studio Ghibli |
| 3 | Cyberpunk Neon | 赛博朋克 | RED Monstro 8K, neon de alto contraste |
| 4 | Chinese Ink Painting | 水墨国风 | ARRI ALEXA Mini LF, render em tinta chinesa |
| 5 | Korean Drama | 韩剧氛围 | Sony VENICE 2, tons quentes e pouca profundidade de campo |
| 6 | Dark Thriller | 暗黑悬疑 | ARRI ALEXA 65, iluminação chiaroscuro |
| 7 | Vintage Hong Kong | 港风复古 | Kodak Vision3, Cooke Anamorphic |
| 8 | Wuxia Epic | 武侠大片 | Panavision DXL2, cenas amplas com névoa |
| 9 | Soft Romance | 甜蜜恋爱 | Canon C500, foco suave e tons quentes |
| 10 | Documentary Real | 纪实写实 | Sony FX6, câmera na mão com luz natural |

O `prompt_suffix` de cada estilo é automaticamente acrescentado ao final de todos os prompts gerados por IA. Você pode customizar ou adicionar novos estilos em `.config/visual_styles.json`.

## Envio de tarefas para o Seedance

O sistema mapeia cada imagem de storyboard (A/B, 1 imagem cada) para 1 tarefa do Seedance. São 2 tarefas por episódio, totalizando 50 tarefas para a obra inteira.

### Estrutura do prompt da tarefa

```
(@DM-002-EP01-A_storyboard.png) is a 6-frame storyboard reference image,
(@林策_ref.png) is the reference appearance for the character “Lin Ce”, (@沈璃_ref.png) is the reference appearance for the character “Shen Li”...

Starting from shot 1, do not display the multi-frame storyboard reference image. Turn the storyboard into a film-grade HD visual production...

DM-002-EP01-A Episode 1 "Breath Tax Era" first half. Plot summary. Atmosphere.

Shot 1 (0.0s-2.5s): Scene description. (@林策_ref.png) Lin Ce action... Lin Ce says: "dialogue" (emotion)
Shot 2 (2.5s-5.0s): ...
...
Shot 6 (12.5s-15.0s): ...
```

### API do Seedance

- Endereço do serviço: `http://localhost:3456`
- Endpoint principal: `POST /api/tasks/push`
- Suporta envio em lote (`tasks` array)
- `realSubmit: false` = modo simulação, `true` = envio real

## Stack de tecnologia

- **Plataforma de skills de IA**: Claude Skills (`.claude/skills/`)
- **Geração de imagem**: Google Gemini (`gemini-2.5-flash-image-preview` / `gemini-3-pro-image-preview`)
- **Geração de vídeo**: Google Veo 2 (`veo-2.0-generate-001`)
- **Envio de tarefas**: pipeline de geração de vídeo do Seedance (HTTP REST API)
- **Ambiente de execução**: Python 3.13+, SDK `google-genai`

## Obras existentes

| Código | Nome | Tipo | Status |
|------|------|------|------|
| DM-001 | "Luzes no Caminho de Volta" | — | Roteirizado |
| DM-002 | "Fúria do Ouro de Carbono" | Ficção científica / finanças / suspense | Roteirizado + storyboard gerado + tarefas geradas |

## Licença

MIT
