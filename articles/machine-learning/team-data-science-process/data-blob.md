---
title: Processar dados de Blobs do Azure com análises avançadas – Processo de Ciência de Dados de Equipe
description: Explorar dados e gerar recursos a partir dos dados armazenados no armazenamento de Blobs do Azure usando análise avançada.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 4c47dfb8b221b6cb4b6237669ecd17c1637107a2
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "76721091"
---
# <a name="process-azure-blob-data-with-advanced-analytics"></a><a name="heading"></a>Processar dados de blob do Azure com análise avançada
Este documento aborda a exploração de dados e a geração de recursos por meio dos dados armazenados no Armazenamento de Blob do Azure. 

## <a name="load-the-data-into-a-pandas-data-frame"></a>Carregar os dados em um quadro de dados Pandas
Para explorar e manipular um conjunto de dados, ele deve ser baixado da origem do blob para um arquivo local que pode ser carregado em um quadro de dados do pandas. Aqui estão as etapas para este procedimento:

1. Baixe os dados do blob do Azure com o seguinte código Python de exemplo usando o serviço BLOB. Substitua a variável no código abaixo pelos valores específicos: 
   
        from azure.storage.blob import BlobService
        import tables
   
        STORAGEACCOUNTNAME= <storage_account_name>
        STORAGEACCOUNTKEY= <storage_account_key>
        LOCALFILENAME= <local_file_name>        
        CONTAINERNAME= <container_name>
        BLOBNAME= <blob_name>
   
        #download from blob
        t1=time.time()
        blob_service=BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
        blob_service.get_blob_to_path(CONTAINERNAME,BLOBNAME,LOCALFILENAME)
        t2=time.time()
        print(("It takes %s seconds to download "+blobname) % (t2 - t1))
2. Leia os dados em um quadro de dados Pandas do arquivo baixado.
   
        #LOCALFILE is the file path    
        dataframe_blobdata = pd.read_csv(LOCALFILE)

Agora você está pronto para explorar os dados e gerar recursos neste conjunto de dados.

## <a name="data-exploration"></a><a name="blob-dataexploration"></a>Exploração de dados
Veja estão alguns exemplos de maneiras de explorar dados usando Pandas:

1. Verificar o número de linhas e colunas 
   
        print 'the size of the data is: %d rows and  %d columns' % dataframe_blobdata.shape
2. Inspecione as primeira ou últimas linhas no conjunto de dados, conforme mostrado a seguir:
   
        dataframe_blobdata.head(10)
   
        dataframe_blobdata.tail(10)
3. Verifique o tipo de dados indicado para cada coluna importada usando o seguinte código de exemplo
   
        for col in dataframe_blobdata.columns:
            print dataframe_blobdata[col].name, ':\t', dataframe_blobdata[col].dtype
4. Verifique as estatísticas básicas para as colunas no conjunto de dados da seguinte maneira
   
        dataframe_blobdata.describe()
5. Veja o número de entradas para cada valor de coluna da seguinte maneira
   
        dataframe_blobdata['<column_name>'].value_counts()
6. Conte os valores ausentes em comparação com o número real de entradas em cada coluna usando o seguinte código de exemplo
   
        miss_num = dataframe_blobdata.shape[0] - dataframe_blobdata.count()
        print miss_num
7. Se você tiver valores ausentes para uma coluna específica nos dados, poderá removê-los da seguinte maneira:
   
        dataframe_blobdata_noNA = dataframe_blobdata.dropna()
        dataframe_blobdata_noNA.shape
   
   Outra maneira de substituir valores ausentes é com a função de modo:
   
        dataframe_blobdata_mode = dataframe_blobdata.fillna({'<column_name>':dataframe_blobdata['<column_name>'].mode()[0]})        
8. Crie um gráfico de histograma usando um número variável de compartimentos para plotar a distribuição de uma variável    
   
        dataframe_blobdata['<column_name>'].value_counts().plot(kind='bar')
   
        np.log(dataframe_blobdata['<column_name>']+1).hist(bins=50)
9. Examine correlações entre variáveis usando um gráfico disperso ou a função interna de correlação
   
        #relationship between column_a and column_b using scatter plot
        plt.scatter(dataframe_blobdata['<column_a>'], dataframe_blobdata['<column_b>'])
   
        #correlation between column_a and column_b
        dataframe_blobdata[['<column_a>', '<column_b>']].corr()

## <a name="feature-generation"></a><a name="blob-featuregen"></a>Geração de recursos
Podemos gerar recursos usando o Python da seguinte maneira:

### <a name="indicator-value-based-feature-generation"></a><a name="blob-countfeature"></a>Geração de Recursos baseada no valor do indicador
Recursos categóricos podem ser criados da seguinte maneira:

1. Inspecionar a distribuição da coluna categórica:
   
        dataframe_blobdata['<categorical_column>'].value_counts()
2. Gerar valores de indicador para cada um dos valores da coluna
   
        #generate the indicator column
        dataframe_blobdata_identity = pd.get_dummies(dataframe_blobdata['<categorical_column>'], prefix='<categorical_column>_identity')
3. Unir a coluna de indicador com o quadro de dados original 
   
            #Join the dummy variables back to the original data frame
            dataframe_blobdata_with_identity = dataframe_blobdata.join(dataframe_blobdata_identity)
4. Remover a própria variável original:
   
        #Remove the original column rate_code in df1_with_dummy
        dataframe_blobdata_with_identity.drop('<categorical_column>', axis=1, inplace=True)

### <a name="binning-feature-generation"></a><a name="blob-binningfeature"></a>Agrupamento da Geração de Recursos
Para gerar recursos compartimentalizados, faça o seguinte:

1. Adicione uma sequência de colunas a ser compartimentalizada a coluna numérica
   
        bins = [0, 1, 2, 4, 10, 40]
        dataframe_blobdata_bin_id = pd.cut(dataframe_blobdata['<numeric_column>'], bins)
2. Converta a compartimentalização em uma sequência de variáveis boolianas
   
        dataframe_blobdata_bin_bool = pd.get_dummies(dataframe_blobdata_bin_id, prefix='<numeric_column>')
3. Por fim, associe as variáveis fictícias ao quadro de dados original
   
        dataframe_blobdata_with_bin_bool = dataframe_blobdata.join(dataframe_blobdata_bin_bool)    

## <a name="writing-data-back-to-azure-blob-and-consuming-in-azure-machine-learning"></a><a name="sql-featuregen"></a>Gravar dados de volta ao blob do Azure e consumi-los no Azure Machine Learning
Depois de explorar os dados e criar os recursos necessários, você pode carregar os dados (amostras ou destacados) em um blob do Azure e consumi-los em Azure Machine Learning usando as seguintes etapas: os recursos adicionais podem ser criados no Azure Machine Learning Studio (clássico) também. 

1. Grave o quadro de dados no arquivo local
   
        dataframe.to_csv(os.path.join(os.getcwd(),LOCALFILENAME), sep='\t', encoding='utf-8', index=False)
2. Carregue os dados para o blob do Azure da seguinte maneira:
   
        from azure.storage.blob import BlobService
        import tables
   
        STORAGEACCOUNTNAME= <storage_account_name>
        LOCALFILENAME= <local_file_name>
        STORAGEACCOUNTKEY= <storage_account_key>
        CONTAINERNAME= <container_name>
        BLOBNAME= <blob_name>
   
        output_blob_service=BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)    
        localfileprocessed = os.path.join(os.getcwd(),LOCALFILENAME) #assuming file is in current working directory
   
        try:
   
        #perform upload
        output_blob_service.put_block_blob_from_path(CONTAINERNAME,BLOBNAME,localfileprocessed)
   
        except:            
            print ("Something went wrong with uploading blob:"+BLOBNAME)
3. Agora, os dados podem ser lidos do blob usando o módulo [Importar Dados][import-data] do Azure Machine Learning, como mostra a tela abaixo:

![blob de leitor][1]

[1]: ./media/data-blob/reader_blob.png


<!-- Module References -->
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/

