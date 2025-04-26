# 🛡️ Black Shark Anti-Cheat Integration Guide

Este repositório contém a documentação necessária para integrar o sistema **Black Shark Anti-Cheat** ao seu projeto de jogo. Abaixo estão os passos detalhados para configuração, estrutura de pastas e modificações de código necessárias.

---

## 📁 Estrutura de Pastas

### `ClientDLL`
- DLLs do cliente.
- ⚠️ **Atenção:** NÃO envie arquivos `.exp`, `.lib` ou `BSPV.bin` aos jogadores.
- `BSPV.bin` é uma **chave privada**, mantenha-a segura no servidor.

### `DependenciesErrors`
- Instaladores de dependências:
  - DirectX
  - Microsoft C++ Runtime
- Inclua esses instaladores para prevenir erros de execução nos clientes.

### `DocumentationSDK`
- Contém a documentação da SDK para consultas e integração avançada.

### `Include`
- Arquivo `BSAnticheat.hpp` com as funções expostas pelo sistema anti-cheat.

### `Server`
- Servidor secundário do anti-cheat (executado pelo cliente).
- Comunicação P2P com o servidor central da Black Shark.

#### Subpastas do `Server/data`:
- `[Block]` – Jogadores banidos.
- `[CRC]` – Verificação de integridade das DLLs.
- `[hacks]` – Identificação de cheats conhecidos.
- `[hwid]` – IDs de hardware dos jogadores.
- `[info]` – Informações gerais dos jogadores.
- `[logs]` – Logs de injeção/detecção.
- `[ss]` – Screenshots coletadas dos jogadores.
- `[update]` – Atualizações futuras da Black Shark.

---

## 🔧 Etapas de Integração

### 1. `WinMain.cpp`

#### Adicione após `WinMain`:

```cpp
#ifdef FINAL_BUILD
    if (!BS_Start()) {
        MessageBox(NULL, "An error has occurred starting Black Shark!", "Black Shark Anticheat", MB_ICONERROR | MB_OK);
        ExitProcess(0);
        return 0;
    }
#endif
```

#### Inclua no topo do arquivo:

```cpp
#define ENABLED_BLACKSHARK 1
#ifdef ENABLED_BLACKSHARK
    #include "C:\\WarZ\\BSAnticheat\\include\\BSAnticheat.hpp"
#endif
```

---

### 2. `FrontendWarZ.cpp`

Após:

```cpp
r3dOutToLog("Acquired base profile data for %f\n", r3dGetTime() - aquireProfileStart);
```

Adicione:

```cpp
#ifdef FINAL_BUILD
    if (!BS_SetParam(NULL, NULL, NULL, gUserProfile.CustomerID)) {
        return FrontEndShared::RET_Diconnected;
    }
#endif
```

Repita a inclusão do cabeçalho como no passo anterior.

---

### 3. `FrontEndShare.h`

Atualize o enum `EResults`:

```cpp
enum EResults {
    RET_Exit = 100,
    RET_JoinGame,
    RET_Diconnected,
    RET_DoubleLogin,
    RET_Banned,
    RET_BannedBSA,
    RET_LoggedIn,
};
```

---

### 4. `Main_Network.cpp`

Após:

```cpp
else if (res == FrontEndShared::RET_Banned) {
    showLoginErrorMsg = gLangMngr.getString("LoginMenu_AccountFrozen");
    goto repeat_the_login;
}
```

Adicione:

```cpp
else if (res == FrontEndShared::RET_BannedBSA) {
    showLoginErrorMsg = gLangMngr.getString("LoginMenu_AccountFrozenBSA");
    goto repeat_the_login;
}
```

---

### 5. 📜 Arquivo de Localização

Adicione ao arquivo de idiomas:

```ini
[UI General]
LoginMenu_AccountFrozen=Login failed.\nAccount is banned.
LoginMenu_AccountFrozenBSA=Login failed.\nA possible cheat bypass was detected by Black Shark.\nYou have been reported to our servers.
```

---

## 🖥️ Integração no Servidor

### `ServerGameLogic.cpp`

Logo após verificar jogadores temporariamente banidos:

```cpp
	// check if that player was kicked before and ban time isn't expired
	{
		TKickedPlayers::iterator it = kickedPlayers_.find(n.CustomerID);
		if(it != kickedPlayers_.end() && r3dGetTime() < it->second)
		{
			PKT_S2C_CustomKickMsg_s n2;
			r3dscpy(n2.msg, "You are temporarily banned from this server");
			p2pSendRawToPeer(peerId, &n2, sizeof(n2));

			PKT_S2C_JoinGameAns_s n;
			n.success   = 0;
			n.playerIdx = 0;
			n.ownerCustomerID = 0;
			p2pSendRawToPeer(peerId, &n, sizeof(n));

			DisconnectPeer(peerId, false, "temporary ban");
			return;
		}
	}
```
Adicione:
```cpp
if (!BS_CheckPlayer(n.CustomerID, "IP-VPS", 3408, "YOUR-KEY"))
{
    PKT_S2C_CustomKickMsg_s n2;
    r3dscpy(n2.msg, "An attempt was made to bypass in BSA. If you continue to try it will take permanent ban!");
    p2pSendRawToPeer(peerId, &n2, sizeof(n2));

    PKT_S2C_JoinGameAns_s n;
    n.success   = 0;
    n.playerIdx = 0;
    n.ownerCustomerID = 0;
    p2pSendRawToPeer(peerId, &n, sizeof(n));

    DisconnectPeer(peerId, false, "bypass result ban");
    return;
}
```

---

## ✅ Considerações Finais

- Mantenha a `BSPV.bin` segura — isso é crucial!
- Sempre teste a integração antes de enviar aos jogadores.
- Mantenha os instaladores de dependências atualizados.
- Use logs e screenshots para análise de casos suspeitos.

---

## 📬 Suporte

Em caso de dúvidas ou suporte técnico, entre em contato com a equipe Black Shark.

---

