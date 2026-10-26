# Perfil GitHub animado

Este projeto gera imagens SVG animadas para um README de perfil: retrato ASCII,
heatmap de contribuições e cards de estatísticas. O processamento das imagens e
a renderização dos SVGs acontecem localmente.

## Privacidade e conexões de rede

- Nenhum script envia dados ao site do artigo nem ao seu autor; não há
  telemetria, pixel de rastreamento ou chamada para esse domínio.
- `fetch_contributions.py` faz uma requisicao HTTP sem autenticacao somente para
  `github.com/users/<usuário>/contributions`. O nome informado em
  `GH_PROFILE_USER` aparece no caminho da URL; o script não envia token, foto ou
  outro dado no corpo da requisição.
- `generate_streak_svg.py` usa somente `data/contributions.json` local. Ele não
  consulta mais APIs de terceiros.
- `prep_photo.py` processa a foto localmente. A primeira utilizacao do `rembg`
  pode baixar o modelo necessário; a foto não é enviada pelo script.
- O workflow opcional abaixo consulta o GitHub e grava os resultados neste
  repositório. Se não quiser esse acesso/atualização automática, desative-o em
  **GitHub > Actions**.

## Passo a passo

### 1. Configure o repositorio e o usuario

Use um repositorio de perfil com o mesmo nome da sua conta GitHub. Abra o
PowerShell na raiz do repositorio e informe o usuario explicitamente:

```powershell
$env:GH_PROFILE_USER = "seu-usuario"
```

O coletor não assume o usuário do artigo nem de outra pessoa. A variável vale
somente para a janela atual do PowerShell.

### 2. Instale as dependencias

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r scripts\requirements.txt
python -m pip install -r scripts\requirements-photo.txt
```

`requirements.txt` contém apenas o necessário para atualizar as estatísticas.
`requirements-photo.txt` instala também as bibliotecas usadas para preparar o
retrato.

### 3. Prepare o retrato e gere o SVG ASCII

Substitua `scripts\source-photo.jpg` pela sua imagem, se desejar, e execute:

```powershell
python scripts\prep_photo.py scripts\source-photo.jpg source-prepped.png
python scripts\make_ascii_svg.py source-prepped.png avi-ascii.svg
```

O primeiro comando remove o fundo, ajusta o contraste e cria a imagem
monocromática local `source-prepped.png`. O segundo transforma essa imagem em
`avi-ascii.svg`. Altere o texto de exemplo no gerador para personalizar o nome
que aparece no desenho.

### 4. Atualize os dados públicos do GitHub e gere os cards

Esta etapa precisa de internet e consulta somente o calendário público do
usuário indicado em `GH_PROFILE_USER`; não requer token:

```powershell
python scripts\fetch_contributions.py
python scripts\render_heatmap_svg.py
python scripts\render_stats_svg.py
python scripts\generate_streak_svg.py
```

Os arquivos gerados são `data\contributions.json`, `contrib-heatmap.svg`,
`stats.svg` e `streak.svg`. O gerador de streak lê o JSON local e falha com uma
mensagem clara se ele estiver ausente ou pertencer a outro usuário.

### 5. Inclua as imagens no README do perfil

Adicione ao README do seu repositorio de perfil os caminhos das imagens que
quiser exibir, por exemplo:

```html
<div align="center">
  <img src="./contrib-heatmap.svg" width="860" alt="Heatmap de contribuições" />
  <br /><br />
  <img src="./avi-ascii.svg" width="370" alt="Retrato em ASCII" />
  <img src="./stats.svg" width="490" alt="Estatísticas do GitHub" />
</div>
```

### 6. Atualização automática (opcional)

`.github/workflows/update-profile-art.yml` executa diariamente e tambem pode
ser acionado manualmente na aba **Actions**. Ele lê o calendário público do
proprietário do repositório, atualiza os SVGs e commita os resultados no próprio
repositório, usando a permissão `contents: write`. Não configure esse workflow
se não quiser que o GitHub Actions faça essas requisições e commits.
