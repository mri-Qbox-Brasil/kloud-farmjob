# kloud-farmjob — Manual

Emprego de agricultor: colha batata, repolho, tomate, grãos de café e laranja nos campos, lave a colheita e venda para o feirante.

---

## Sumário

1. [Dependências](#dependências)
2. [Instalação](#instalação)
3. [Configuração geral](#configuração-geral)
4. [Campos de colheita](#campos-de-colheita)
5. [Árvores](#árvores)
6. [Lavagem da colheita](#lavagem-da-colheita)
7. [Lojas](#lojas)
8. [Entrypoints para outros recursos](#entrypoints-para-outros-recursos)
9. [Localização](#localização)
10. [Estrutura de arquivos](#estrutura-de-arquivos)

---

## Dependências

| Recurso | Obrigatório | Observação |
|---|---|---|
| `ox_lib` | Sim | Declarado em `dependencies`. Zonas, target, progressbar, skill check, menus e locale |
| `oxmysql` | Sim | Declarado em `dependencies` e carregado nos `server_scripts` |
| `qb-core` **ou** `qbx-core` **ou** `es_extended` | Sim | Framework. Detectado em runtime por `GetResourceState` nos arquivos de `framework/` |
| `ox_target` **ou** `qb-target` | Sim | Target. Detectado em runtime pelos arquivos de `target/` |
| `ox_inventory` **ou** `qb-inventory` **ou** `qs-inventory` | Sim | Inventário. Detectado em runtime pelos arquivos de `inventory/` |

O recurso avisa no console de inicialização (`shared/checks.lua`) se não encontrar framework ou target ativos.

---

## Instalação

1. Copie a pasta `kloud-farmjob` para `resources/`.
2. Adicione ao `server.cfg`, garantindo que framework, target e inventário subam **antes**:
   ```
   ensure ox_lib
   ensure kloud-farmjob
   ```
3. Copie as imagens de `install/images/` para a pasta de imagens do seu inventário (em `ox_inventory`, `web/images/`).
4. Cadastre os itens no seu inventário. São necessários:

   | Item | Uso |
   |---|---|
   | `trowel` | Ferramenta exigida no campo de batata (com durabilidade e chance de quebra) |
   | `shovel` | Vendido na loja de agricultura |
   | `dirty_potato`, `dirty_cabbage`, `dirty_tomato`, `dirty_coffee_beans` | Resultado da colheita, antes da lavagem |
   | `potato`, `cabbage`, `tomato`, `coffee_beans` | Resultado da lavagem; itens vendáveis |
   | `orange` | Colhida já limpa, direto da árvore |

   O `README.md` do recurso traz os blocos prontos de `qb-core/shared/items.lua` e de `ox_inventory/data/items.lua`.
5. Não há SQL a importar — o recurso usa o `oxmysql` apenas via wrappers do framework, sem tabela própria.

---

## Configuração geral

Arquivo: `shared/config.lua`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `KloudDev.Debug` | bool | Sim | Desenha as zonas e os targets em modo debug |
| `KloudDev.ImagePath` | string | Sim | Caminho NUI das imagens dos itens exibidas nos menus de loja. Padrão: `https://cfx-nui-ox_inventory/web/images/` |
| `KloudDev.DrawSprite` | bool | Sim | Indicador de sprite do `ox_target` |
| `KloudDev.DrawText` | string | Sim | Sistema de TextUI: `"ox"` ou `"qb"` |
| `KloudDev.Menu` | string | Sim | Sistema de menu: `"ox"` ou `"qb"` |
| `KloudDev.Notify` | string | Sim | Sistema de notificação: `"ox"`, `"qb"`, `"esx"` ou `"ps"` |
| `KloudDev.NotifyPos` | string | Sim | Posição da notificação do `ox_lib` (`top`, `center-left`, `bottom-right`, …). Ignorado nos demais sistemas |
| `KloudDev.Progress` | string | Sim | Estilo da progressbar: `"ox-bar"` ou `"ox-circle"` |
| `KloudDev.ProgressCirclePos` | string | Sim | Posição da progressbar circular: `"middle"` ou `"bottom"` |
| `KloudDev.DrawTextAlignment` | string | Sim | Alinhamento do TextUI: `"top"`, `"right"` ou `"left"` |

---

## Campos de colheita

Definidos em `KloudDev.Locations` (`shared/location_config.lua`). Cada campo aceita:

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `coords` | `vec4` | Sim | Centro da zona |
| `zoneType` | string | Sim | `"sphere"` ou `"box"` |
| `zoneRadius` | number | Sim (sphere) | Raio da zona esférica |
| `zoneSize` | `vec3` | Sim (box) | Dimensões da zona em caixa |
| `prop` | string | Sim | Modelo do prop plantado espalhado pela zona |
| `job` | string \| false | Sim | `false` libera para todos; uma string exige o job correspondente |
| `action.type` | string | Sim | `"progress"` (barra de progresso) ou `"skillCheck"` (minigame do `ox_lib`) |
| `action.progressDuration` | number (ms) | Sim | Duração da progressbar quando `type = "progress"` |
| `action.skillCheckDifficulty` | array | Sim | Dificuldades do skill check: `"easy"`, `"medium"`, `"hard"` |
| `action.skillCheckInputs` | array | Sim | Teclas do skill check |
| `max` | number | Sim | Quantidade de props spawnados na zona |
| `target.label` / `target.icon` | string | Sim | Texto e ícone da opção de target |
| `anim.scenario` | string \| nil | Sim | Scenario do GTA V; se `nil`, usa `dict`/`clip` |
| `anim.dict` / `anim.clip` | string \| nil | Sim | Dicionário e clip de animação |
| `anim.upperBody` | bool | Sim | Restringe a animação ao tronco |
| `item.require.enable` | bool | Sim | Se a colheita exige uma ferramenta no inventário |
| `item.require.item` | string | Sim | Nome da ferramenta exigida |
| `item.require.durability.subtract` | bool | Sim | Se a ferramenta perde durabilidade |
| `item.require.durability.amount` | number | Sim | Quanto de durabilidade é descontado |
| `item.require.durability.chance` | number | Sim | Chance percentual do desconto acontecer |
| `item.require.breaking.enabled` | bool | Sim | Se a ferramenta pode quebrar |
| `item.require.breaking.chance` | number | Sim | Chance percentual de quebra |
| `item.name` / `item.label` | string | Sim | Item recebido pela colheita |
| `item.min` / `item.max` | number | Sim | Faixa de quantidade recebida por colheita |
| `blip` | tabela | Sim | `enabled`, `label`, `sprite`, `scale`, `colour` |

Os campos que vêm no config são `potato` (exige `trowel`), `cabbage`, `tomato` e `coffee_beans`.

> A durabilidade e a quebra da ferramenta são resolvidas no servidor lendo `requireItem.randomBreak` e `requireItem.breakChance` — chaves que **não existem** na estrutura `item.require` do config, que usa `breaking.enabled` e `breaking.chance`. Na prática a ferramenta perde durabilidade, mas nunca quebra.

---

## Árvores

Definidas em `KloudDev.Trees`. Usam os mesmos campos dos campos de colheita, com dois adicionais:

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `treeBoxes` | array de `vec3` | Sim | Coordenadas das árvores individuais que recebem target. Não há spawn de props: as árvores são as do mapa |
| `cooldown` | number (s) | Sim | Tempo até a árvore poder ser colhida de novo |
| `zonePoints` | array de `vec3` | Sim (poly) | Vértices da zona, quando `zoneType = "poly"` |

A única árvore configurada é `orange`, que entrega o item `orange` já limpo (não passa pela lavagem).

---

## Lavagem da colheita

Definida em `KloudDev.WashLocations`. Nos pontos de lavagem, pressione `E` para converter os itens sujos em limpos.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `blip` | tabela | Sim | `enabled`, `label`, `sprite`, `scale`, `colour` |
| `duration` | number (ms) | Sim | Duração **por item**. Padrão: `3000` (10 itens = 30 segundos) |
| `maxWash` | number | Sim | Máximo de itens por lavagem. Padrão: `20` |
| `anim` | tabela | Sim | Mesma estrutura de `anim` dos campos (`scenario`, `dict`, `clip`, `upperBody`) |
| `items` | array de pares | Sim | `{itemNecessario, itemResultante}` |
| `coords` | array de `vec4` | Sim | Pontos de lavagem no mundo |

---

## Lojas

Definidas em `KloudDev.Shops`, com dois NPCs. Cada um usa `blip`, `coords` (array de `vector4`), `pedModels` (sorteado) e `prices` (array de `{item, preço}`).

| Loja | Papel | Padrão |
|---|---|---|
| `sell` | Feirante — compra a colheita do jogador. O dinheiro entra na conta `cash` | `potato`, `tomato`, `orange`, `cabbage`, `coffee_beans` a $50 |
| `shop` | Loja da Agricultura — vende itens ao jogador, debitando da conta `cash` | `shovel` $100, `trowel` $50, sementes/colheitas a $75 |

---

## Entrypoints para outros recursos

O recurso não expõe exports nem comandos. Os eventos abaixo são de cliente e podem ser disparados por outro recurso, mas todos dependem do estado interno de zona (`_G.CurrentZone`).

```lua
-- Abre o menu de venda do feirante
TriggerEvent('kloud-farm:client:openSell')

-- Abre o menu da loja de agricultura
TriggerEvent('kloud-farm:client:openShop')

-- Inicia a lavagem da colheita
TriggerEvent('kloud-farm:client:wash')
```

Os callbacks de servidor (`kloud-farm:callback:canStart`, `:uprooted`, `:washed`, `:sellItem`, `:buyItem`, `:canCarry`) são internos: recebem os dados da zona vindos do cliente e não são feitos para chamada externa.

---

## Localização

As strings são traduzidas via `ox_lib` locale. Os arquivos ficam em `locales/`:

- `en.json` — inglês
- `pt-br.json` — português do Brasil

O locale ativo é definido pela convar `ox:locale` no `server.cfg`:

```
setr ox:locale "pt-br"
```

Alguns textos **não** passam pelo sistema de locale e estão escritos direto no `shared/location_config.lua`: os `target.label` dos campos ("Uproot", "Pick Orange"), os `blip.label` e os `item.label`. Para traduzi-los, edite o config.

---

## Estrutura de arquivos

```
kloud-farmjob/
├── init.lua                      — cria zonas, blips, targets e NPCs na inicialização
├── server.lua                    — callbacks de colheita, lavagem, compra e venda
├── shared/
│   ├── config.lua                — debug, UI (notify/progress/target/menu) e caminho das imagens
│   ├── location_config.lua       — campos, árvores, pontos de lavagem e lojas
│   ├── animations.lua            — dicionários e clips usados pelos NPCs e ações
│   ├── checks.lua                — avisa no console se faltar framework ou target
│   └── utils.lua                 — wrappers de TextUI, progressbar, notificação e skill check
├── modules/
│   ├── field/
│   │   ├── events.lua            — eventos de colher prop, colher árvore, cooldown e lavar
│   │   └── functions.lua         — spawn/limpeza de props, animações e fluxo da lavagem
│   └── shops/
│       └── events.lua            — menus de venda e de compra
├── framework/
│   ├── client/{qb,esx}.lua       — dados e job do jogador por framework
│   └── server/{qb,esx}.lua       — dinheiro e player por framework
├── inventory/
│   ├── client/{ox_inventory,qb-inventory,qs-inventory}.lua
│   └── server/{ox_inventory,qb-inventory,qs-inventory}.lua
├── target/
│   ├── ox_target.lua             — wrappers de target do ox_target
│   └── qb-target.lua             — wrappers de target do qb-target
├── locales/
│   ├── en.json
│   └── pt-br.json
├── install/images/               — imagens dos itens (copiar para o inventário)
└── fxmanifest.lua
```
