---
title: incluir arquivo
description: incluir arquivo
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 0e009354e66ab13cdb9fbc3cf9e4b37e904bdfd1
ms.sourcegitcommit: fad3aaac5af8c1b3f2ec26f75a8f06e8692c94ed
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/27/2020
ms.locfileid: "67171752"
---
Você pode verificar se a conexão foi bem-sucedida usando o comando [az network vpn-connection show](/cli/azure/network/vpn-connection). No exemplo, "--name" refere-se ao nome da conexão que você deseja testar. Quando a conexão ainda está sendo estabelecida, seu status de conexão mostra "Conectando". Quando a conexão for estabelecida, o status será alterado para "Conectado".

```azurecli
az network vpn-connection show --name VNet1toSite2 --resource-group TestRG1
```
