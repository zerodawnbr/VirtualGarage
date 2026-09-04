# 🚘 Garagem Virtual

O **ZDBR Virtual Garage** é um sistema de armazenamento definitivo, inteligente e universal para servidores de DayZ. Ele contorna as limitações de física e herança do motor Enfusion para oferecer suporte total a **carros, caminhões, barcos, helicópteros, aviões, quadriciclos e bicicletas**

Qualquer veículo que herde das classes nativas `CarScript`, `Boat` ou `Helicopter` é automaticamente identificado, rastreado e armazenado pelo sistema.

---

## ✨ Features Principais

* **Limite Configurável por Jogador:** Evite a superlotação do servidor definindo um limite global de veículos por usuário.
* **Telemetria Automotiva Real-Time:** O painel extrai dados vivos direto da engine do jogo, exibindo:
  * ⛽ Combustível (Litros exatos)
  * 🛢️ Óleo, Radiador e Fluido de Freio
  * 🔋 Status da Bateria (Porcentagem baseada em energia)
  * ⚙️ Integridade do Bloco do Motor (OK ou Danificado)
  * 👥 Lotação Máxima (Assentos/CrewSize)
  * 🎒 Slots Livres e Itens Salvos no Porta-Malas
* **Salva Tudo:** O sistema varre todos os *attachments* (peças, portas, rodas) e varre recursivamente todo o *cargo* (loot dentro de mochilas dentro do porta-malas), mantendo quantidade, líquidos e saúde de cada item.

---

## 🧠 Heurística de Interface e Categorização

O menu principal possui um sistema de *Bucketing* que organiza seus veículos na tela de forma rígida e colorida, priorizando a usabilidade do jogador:

1. 🔵 **Veículos Livres (Status 1):** Veículos sem dono. Só aparecem na sua lista se você estiver a menos de 25 metros deles, evitando que a tela fique poluída com sucatas espalhadas pela cidade.
2. 🟠 **Seus Veículos Perto (Status 2):** Seus veículos fora da garagem, dentro de um raio de interação.
3. 🔴 **Veículos Perdidos (Status 3):** Seus veículos que estão espalhados pelo mapa (rastreados globalmente por um radar de 1000m).
4. 🟢 **Guardados (Status 0):** Veículos armazenados em segurança no banco de dados.

---

## 🗺️ Sistema de GPS e Resgate Físico

O painel de Mapa transforma o mod em um verdadeiro GPS automotivo.

* **Linha Guia Dinâmica:** Traça uma rota reta (usando Canvas e algoritmo Cohen-Sutherland para não vazar da tela) entre você e o veículo perdido.
* **Ícones Inteligentes:** Injeção automática de ícones (`.paa`) baseados na classe: Helicóptero, Barco, Moto ou Carro.
* **Proteção de Resgate (Anti-Explosão):**
  * 🚗 **Carros/Caminhões:** São teletransportados com segurança para o seu lado (com checagem de colisão `IsBoxColliding`).
  * 🚁 **Aeronaves e Embarcações:** Por possuírem hitboxes colossais, o botão de resgate se transforma no modo "Guardar Remoto", sugando o veículo do mundo direto para sua garagem virtual para que você o retire em um espaço seguro.

---

## 🛡️ Painel Administrativo de Controle

Sistema *In-Game* acionado por tecla de atalho (`UAVirtualGarageAdminToggle`), eliminando a necessidade de editar arquivos JSON manualmente pelo servidor FTP.

* **Gestão Global:** Altere o comando de chat, o limite de veículos e a logomarca do UI diretamente pelo painel.
* **Pesquisa de Jogadores:** Localize arquivos de usuários pelo Steam64 ID ou Nome.
* **Controle de Frota:** Veja todos os carros de um jogador específico.
* **Ferramentas de Seguradora:** 
  * Ative/desative a flag de Seguro e reparo automático de Saúde do veículo selecionado.
  * Edite datas de início e expiração de apólices diretamente via *EditBox* com validação de formato.
  * Apague um veículo permanentemente com um clique.

---

## 🗄️ Estrutura de Banco de Dados (Arquivos JSON)

A persistência de dados do **ZDBR Virtual Garage** é modular. O sistema divide as responsabilidades em três níveis de arquivos `.json` para garantir máxima segurança contra corrupções e facilidade de manipulação para os administradores.

---

### 1. Configuração Global (`config.json`)
**Diretório:** `$profile:ZeroDawnBRCoreTools/VirtualGarage/config.json`
Responsável pelas regras gerais do mod no servidor. Pode ser atualizado *in-game* via Painel Admin.

| Atributo | Tipo | Descrição |
| :--- | :--- | :--- |
| `m_version` | Inteiro | Versão atual da estrutura do arquivo de configuração. |
| `m_description` | String | Descrição legível do propósito do sistema. |
| `m_enable` | Inteiro | Chave mestre (1 = Ligado / 0 = Desligado) para o funcionamento do mod. |
| `m_command` | String | Comando de chat utilizado pelos administradores para forçar o recarregamento (`\garagemreload`). |
| `m_vehicleLimitPlayer`| Inteiro | Quantidade máxima de veículos que um único jogador pode registrar no servidor. |
| `m_logo` | String | Caminho local (path) para a renderização do banner principal nas interfaces gráficas. |

---

### 2. Perfil do Jogador (`config.json`)
**Diretório:** `$profile:ZeroDawnBRCoreTools/VirtualGarage/Players/<Steam64_ID>/config.json`
Funciona como o "documento de identidade" do jogador. Ele não salva as peças do carro, apenas mapeia o que pertence ao jogador e o status da frota.

| Atributo | Tipo | Descrição |
| :--- | :--- | :--- |
| `m_SteamId` | String | Steam64 ID exclusivo do proprietário da garagem. |
| `m_Vehicles` | Array | Lista (matriz) contendo todos os veículos vinculados a este jogador. |
| ↳ `m_VehiclesId` | Inteiro | ID matemático único do veículo, usado para cruzar os dados com o arquivo físico do carro. |
| ↳ `m_Stored` | Inteiro | Define fisicamente se a carcaça está guardada na base de dados (1) ou solta pelo mapa (0). |
| ↳ `m_insurance` | Inteiro | Apólice de seguro: 1 = Ativo (recupera fluidos de graça) / 0 = Inativo. |
| ↳ `m_initialExpirationDate`| String | Data de emissão/renovação do seguro no formato AAAA-MM-DD. |
| ↳ `m_finalExpirationDate` | String | Data de vencimento do seguro no formato AAAA-MM-DD. |
| ↳ `m_type` | String | Nome exato da classe do veículo na engine (Ex: `Sedan_02_Red`). |
| ↳ `m_healthItems` | Inteiro | Pacote mecânico: 1 = Restaura 100% da vida da carcaça e anexos ao retirar da garagem / 0 = Mantém dano original. |

---

### 3. Telemetria do Veículo (`[ID_DO_VEICULO].json`)
**Diretório:** `$profile:ZeroDawnBRCoreTools/VirtualGarage/Players/<Steam64_ID>/<m_VehiclesId>.json`
A "alma" do veículo. Só é lido ou reescrito quando o jogador interage diretamente com ele. Salva dados absolutos de posição, mecânica e todo o porta-malas.

| Atributo | Tipo | Descrição |
| :--- | :--- | :--- |
| `UniqueID` | Inteiro | Confirmação de segurança do ID do veículo. |
| `Type` | String | Classe de spawn do veículo (ignora sufixos como `_ruined` se tiver seguro). |
| `OwnerID` | String | GUID ofuscado gerado pelo motor do jogo. |
| `OwnerCarID` / `Owner` | String | Steam64 ID de segurança e o nome em texto legível do dono (Ex: `Sentinela`). |
| `Position` | Array | Matriz `[X, Y, Z]` registrando a coordenada espacial milimétrica de onde o veículo foi guardado/rastreado. |
| `Fuel` / `Oil` / `Coolant` / `Brake`| Float | Volume absoluto de fluidos vitais mapeados no exato milissegundo do armazenamento. |
| `EngineHit` | Inteiro | Flag booleana que detecta se o bloco principal do motor estava fundido/danificado (1) ou funcional (0). |
| `WheelsPresent` | Inteiro | Telemetria que conta as instâncias de pneus encaixados fisicamente no chassi. |
| `CrewSize` | Inteiro | Capacidade máxima de transporte humano da classe. |
| `Attachments` | Array | A estrutura de Árvore (`ItemNode`) que gerencia portas, rodas e todo o *loot*. Veja a tabela abaixo. |

---

#### 📦 A Árvore de Anexos e Cargas (`Attachments`)
O sistema utiliza um *preorder traversal* recursivo. Isso significa que ele salva os itens principais e, se houver mochilas, caixas ou maçaricos, salva os itens dentro deles de forma encadeada.

| Parâmetro de Nó (`ItemNode`) | Descrição e Comportamento da Engine |
| :--- | :--- |
| `Type` / `Parent` | A classe do item (ex: `CarBattery`) e a classe do local onde ele está preso (ex: `Sedan_02_Red`). |
| `Health` | Dano base em Float (ex: 200.0) a ser restaurado ao fazer o spawn. |
| `Quantity` / `AmmoCount` | Unidades empilháveis (ex: massa epóxi) ou balas inseridas na câmara/carregador. |
| `Energy` | Carga elétrica restante no componente (vital para `TruckBattery` e `CarBattery`). |
| `LiquidType` | ID de matriz de líquido. Ex: `8192` (Gasolina) guardada em um `CanisterGasoline` dentro do porta-malas. |
| `SlotType` | Define a regra física: `2` (Anexado visualmente no veículo) ou `3` (Jogado solto no porta-malas/Cargo). |
| `AttSlot` / `SlotIdx` | IDs numéricos de *hash* que dizem ao jogo em qual dobradiça exata colocar a porta ou a roda. |
| `AttRow` / `AttCol` / `AttFlip` | Matrix de posicionamento geométrico para o *Tetris* do inventário do jogo (Linha x Coluna e Rotação do item). |
| `Children` | Sub-Matriz de itens. Por exemplo, um Botijão de Gás Grande (`LargeGasCanister`) salvo dentro de um Maçarico (`Blowtorch`) que está dentro do carro. |
