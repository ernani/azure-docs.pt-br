---
title: Copiar dados do Amazon S3 para o armazenamento do Azure usando o AzCopy | Microsoft Docs
description: Transferir dados com o AzCopy e os buckets do Amazon S3
services: storage
author: normesta
ms.service: storage
ms.topic: conceptual
ms.date: 01/13/2020
ms.author: normesta
ms.subservice: common
ms.openlocfilehash: a3180593eaf8c01c772fd761d88b5f5b9f7657ee
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "75941499"
---
# <a name="copy-data-from-amazon-s3-to-azure-storage-by-using-azcopy"></a>Copiar dados do Amazon S3 para o armazenamento do Azure usando o AzCopy

AzCopy é um utilitário de linha de comando que você pode usar para copiar BLOBs ou arquivos de ou para uma conta de armazenamento. Este artigo ajuda você a copiar objetos, diretórios e buckets de Amazon Web Services (AWS) S3 para o armazenamento de BLOBs do Azure usando o AzCopy.

## <a name="choose-how-youll-provide-authorization-credentials"></a>Escolha como você fornecerá credenciais de autorização

* Para autorizar com o armazenamento do Azure, use Azure Active Directory (AD) ou um token de assinatura de acesso compartilhado (SAS).

* Para autorizar com o AWS S3, use uma chave de acesso do AWS e uma chave de acesso secreta.

### <a name="authorize-with-azure-storage"></a>Autorizar com o armazenamento do Azure

Consulte o artigo [introdução ao AzCopy](storage-use-azcopy-v10.md) para baixar o AzCopy e escolha como você fornecerá credenciais de autorização para o serviço de armazenamento.

> [!NOTE]
> Os exemplos neste artigo pressupõem que você autenticou sua identidade usando o `AzCopy login` comando. Em seguida, o AzCopy usa sua conta do Azure AD para autorizar o acesso aos dados no armazenamento de BLOBs.
>
> Se você preferir usar um token SAS para autorizar o acesso a dados BLOB, poderá acrescentar esse token à URL do recurso em cada comando AzCopy.
>
> Por exemplo: `https://mystorageaccount.blob.core.windows.net/mycontainer?<SAS-token>`.

### <a name="authorize-with-aws-s3"></a>Autorizar com AWS S3

Reúna sua chave de acesso do AWS e a chave de acesso secreta e defina as variáveis de ambiente a seguir:

| Sistema operacional | Comando  |
|--------|-----------|
| **Windows** | `set AWS_ACCESS_KEY_ID=<access-key>`<br>`set AWS_SECRET_ACCESS_KEY=<secret-access-key>` |
| **Linux** | `export AWS_ACCESS_KEY_ID=<access-key>`<br>`export AWS_SECRET_ACCESS_KEY=<secret-access-key>` |
| **MacOS** | `export AWS_ACCESS_KEY_ID=<access-key>`<br>`export AWS_SECRET_ACCESS_KEY=<secret-access-key>`|

## <a name="copy-objects-directories-and-buckets"></a>Copiar objetos, diretórios e buckets

O AzCopy usa o [bloco Put da API de URL](https://docs.microsoft.com/rest/api/storageservices/put-block-from-url) , para que os dados sejam copiados diretamente entre AWS S3 e servidores de armazenamento. Essas operações de cópia não usam a largura de banda de rede do seu computador.

> [!IMPORTANT]
> Esse recurso está atualmente na visualização. Se você decidir remover dados de seus buckets S3 após uma operação de cópia, certifique-se de verificar se os dados foram copiados corretamente para sua conta de armazenamento antes de remover os dados.

> [!TIP]
> Os exemplos nesta seção incluem argumentos de caminho com aspas simples (' '). Use aspas simples em todos os shells de comando, exceto pelo shell de comando do Windows (cmd. exe). Se você estiver usando um shell de comando do Windows (cmd. exe), coloque argumentos de caminho com aspas duplas ("") em vez de aspas simples (' ').

 Esses exemplos também funcionam com contas que têm um namespace hierárquico. O [acesso de vários protocolos no data Lake Storage](../blobs/data-lake-storage-multi-protocol-access.md) permite que você use a mesma sintaxe de`blob.core.windows.net`URL () nessas contas. 

### <a name="copy-an-object"></a>Copiar um objeto

Use a mesma sintaxe de URL`blob.core.windows.net`() para contas que têm um namespace hierárquico.

|    |     |
|--------|-----------|
| **Sintaxe** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>/<object-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<blob-name>'` |
| **Exemplo** | `azcopy copy 'https://s3.amazonaws.com/mybucket/myobject' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myblob'` |
| **Exemplo** (namespace hierárquico) | `azcopy copy 'https://s3.amazonaws.com/mybucket/myobject' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myblob'` |

> [!NOTE]
> Os exemplos neste artigo usam URLs de estilo de caminho para buckets AWS S3 (por exemplo `http://s3.amazonaws.com/<bucket-name>`:). 
>
> Você também pode usar URLs de estilo hospedado virtual também (por exemplo: `http://bucket.s3.amazonaws.com`). 
>
> Para saber mais sobre hospedagem virtual de buckets, confira [Hospedagem virtual de buckets]]https://docs.aws.amazon.com/AmazonS3/latest/dev/VirtualHosting.html)(.

### <a name="copy-a-directory"></a>Copiar um diretório

Use a mesma sintaxe de URL`blob.core.windows.net`() para contas que têm um namespace hierárquico.

|    |     |
|--------|-----------|
| **Sintaxe** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>/<directory-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive=true` |
| **Exemplo** | `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |
| **Exemplo** (namespace hierárquico)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

### <a name="copy-a-bucket"></a>Copiar um Bucket

Use a mesma sintaxe de URL`blob.core.windows.net`() para contas que têm um namespace hierárquico.

|    |     |
|--------|-----------|
| **Sintaxe** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>' --recursive=true` |
| **Exemplo** | `azcopy copy 'https://s3.amazonaws.com/mybucket' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive=true` |
| **Exemplo** (namespace hierárquico)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

### <a name="copy-all-buckets-in-all-regions"></a>Copiar todos os buckets em todas as regiões

Use a mesma sintaxe de URL`blob.core.windows.net`() para contas que têm um namespace hierárquico.

|    |     |
|--------|-----------|
| **Sintaxe** | `azcopy copy 'https://s3.amazonaws.com/' 'https://<storage-account-name>.blob.core.windows.net' --recursive=true` |
| **Exemplo** | `azcopy copy 'https://s3.amazonaws.com' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |
| **Exemplo** (namespace hierárquico)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

### <a name="copy-all-buckets-in-a-specific-s3-region"></a>Copiar todos os buckets em uma região S3 específica

Use a mesma sintaxe de URL`blob.core.windows.net`() para contas que têm um namespace hierárquico.

|    |     |
|--------|-----------|
| **Sintaxe** | `azcopy copy 'https://s3-<region-name>.amazonaws.com/' 'https://<storage-account-name>.blob.core.windows.net' --recursive=true` |
| **Exemplo** | `azcopy copy 'https://s3-rds.eu-north-1.amazonaws.com' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |
| **Exemplo** (namespace hierárquico)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

## <a name="handle-differences-in-object-naming-rules"></a>Tratar diferenças nas regras de nomenclatura de objeto

O AWS S3 tem um conjunto diferente de convenções de nomenclatura para nomes de Bucket em comparação com os contêineres de blob do Azure. Você pode ler sobre eles [aqui](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules). Se você optar por copiar um grupo de buckets para uma conta de armazenamento do Azure, a operação de cópia poderá falhar devido a diferenças de nomenclatura.

O AzCopy lida com dois dos problemas mais comuns que podem surgir; buckets que contêm pontos e buckets que contêm hifens consecutivos. Os nomes de buckets S3 AWS podem conter pontos e hifens consecutivos, mas um contêiner no Azure não pode. AzCopy substitui os períodos por hifens e hifens consecutivos por um número que representa o número de hifens consecutivos (por exemplo: `my----bucket` um `my-4-bucket`Bucket chamado se torna. 

Além disso, à medida que AzCopy copia arquivos, ele verifica se há colisões de nomenclatura e tenta resolvê-los. Por exemplo, se houver `bucket-name` Buckets com o nome e `bucket.name`, AzCopy resolverá um Bucket chamado `bucket.name` primeiro para `bucket-name` e, em seguida, `bucket-name-2`para.

## <a name="handle-differences-in-object-metadata"></a>Tratar diferenças nos metadados do objeto

O AWS S3 e o Azure permitem diferentes conjuntos de caracteres nos nomes das chaves de objeto. Você pode ler sobre os caracteres que o AWS S3 usa [aqui](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMetadata.html#object-keys). No lado do Azure, as chaves de objeto de blob aderem às regras de nomenclatura para [identificadores C#](https://docs.microsoft.com/dotnet/csharp/language-reference/).

Como parte de um comando `copy` AzCopy, você pode fornecer um valor para o `s2s-invalid-metadata-handle` sinalizador opcional que especifica como você gostaria de tratar os arquivos em que os metadados do arquivo contêm nomes de chave incompatíveis. A tabela a seguir descreve cada valor de sinalizador.

| Valor do sinalizador | Descrição  |
|--------|-----------|
| **ExcludeIfInvalid** | (Opção padrão) Os metadados não estão incluídos no objeto transferido. AzCopy registra um aviso. |
| **FailIfInvalid** | Os objetos não são copiados. AzCopy registra um erro e inclui esse erro na contagem de falhas que aparece no resumo da transferência.  |
| **RenameIfInvalid**  | AzCopy resolve a chave de metadados inválida e copia o objeto para o Azure usando o par de valor de chave de metadados resolvido. Para saber exatamente quais etapas o AzCopy leva para renomear as chaves de objeto, consulte a seção [como o AzCopy renomeia as chaves de objeto](#rename-logic) abaixo. Se AzCopy não puder renomear a chave, o objeto não será copiado. |

<a id="rename-logic" />

### <a name="how-azcopy-renames-object-keys"></a>Como o AzCopy renomeia as chaves de objeto

AzCopy executa estas etapas:

1. Substitui caracteres inválidos por ' _ '.

2. Adiciona a cadeia `rename_` de caracteres ao início de uma nova chave válida.

   Essa chave será usada para salvar o **valor**de metadados original.

3. Adiciona a cadeia `rename_key_` de caracteres ao início de uma nova chave válida.
   Essa chave será usada para salvar a **chave**inválida de metadados originais.
   Você pode usar essa chave para tentar recuperar os metadados no lado do Azure, uma vez que a chave de metadados é preservada como um valor no serviço de armazenamento de BLOBs.

## <a name="next-steps"></a>Próximas etapas

Encontre mais exemplos em qualquer um destes artigos:

- [Introdução ao AzCopy](storage-use-azcopy-v10.md)

- [Transferir dados com o AzCopy e o Armazenamento de Blobs](storage-use-azcopy-blobs.md)

- [Transferir dados com o AzCopy e o Armazenamento de Arquivos](storage-use-azcopy-files.md)

- [Configurar, otimizar e solucionar problemas do AzCopy](storage-use-azcopy-configure.md)