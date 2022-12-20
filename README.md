# ESTUDO SOBRE O IMPACTO DE DADOS ESPARSOS EM MODELOS BASEADOS EM ÁRVORE DE DECISÃO NA ÁREA DE FLUXOS DE TRANSPORTE VIÁRIO

## Base de dados

A base de dados deste experimento foi extraída da ferramenta de simulação PTV Visum e reflete, em suas linhas, diversos caminhos que partem de um mesmo lugar numa malha viária simulada. As colunas da base são de informações relevantes a este caminho, como o tamnho do trajeto, tempo de locomoção com e sem trânsito e também referente à velocidade média.

A base possui dois tipos de dados:

Paths: dados relacionados a um trajeto inteiro, de um ponto A a um ponto B
Links: dados relacionados a trechos de um trajeto

Obs: as informações sobre links foram adicionadas à tabela de resultados da simulação, dentro da própria ferramenta.

Obs2: a extração da base de dados da ferramenta é feita atráves de Excel e conversão para CSV em Python.

A base possui ao todo 129 linhas e 28 colunas, e suas colunas são as seguintes:

DestZoneNo -> Numero da zona de destino [int64]

(Removida) Path_Volume -> Cálculo de volume de veículos considerando o path (caminho inteiro) [float64]

Path_UnloadedTime -> Tempo de locomoção no path considerando as vias sem transito (calculo mais simplificado considerando a velocidade média da via e a distância) [float64]

Path_UnloadedSpeed ->  Velocidade de locomoção no path considerando as vias sem transito [int64]

Path_LoadedTime -> Tempo de locomoção no path considerando as vias com transito (vem ao rodar a simulação) [float64]

Path_LoadedSpeed -> Velocidade de locomoção no path considerando as vias com transito [int64]

Links_Count -> Quantidade de links que compõe o path [int64]

Links_(Min, Max, Avg, Sum)CarCapacity -> Colunas referente à capacidade de carros nos links/vias [int64]

Links_(Min, Max, Avg, Sum)Length -> Colunas referente ao tamanho das vias [float64]

Links_(Min, Max, Avg, Sum)OfCars -> Colunas referente à quantidade de carros nos links/vias [int64]

Links_(Min, Max, Avg)LoadedSpeed -> Colunas referente à velocidade nas vias/links considerando o transito [int64]

Links_(Min, Max, Avg, Sum)UnloadedTime -> Colunas referente ao tempo de locomoção nas vias/links não considerando o transito [int64, float64]

Links_(Min, Max, Avg, Sum)LoadedTime -> Colunas referente ao tempo de locomoção nas vias/links considerando o transito [int64, float64]

Colunas excluídas:
OrigZoneNo -> Número de origem (sempre 10)  [int64]

Index -> Repetitivo e considera os pares de origem destino, não vi utilidade (a confirmar) [int64]

Links_FromNodeNo -> Lista de nós de partida (essa coluna e a abaixo podem ter alguma utilidade, mas não enxerguei ainda) [object]

Links_ToNodeNo -> Lista de nós de Chegada [object]

### Tratamento da base

Etapa inicial composta apenas de tratamento de string e conversão de colunas, que inicialmente eram do tipo string e passam a ser do tipo numérico, antes disso, removendo caracteres como “km/h” e “min”, por exemplo. Porém, há tratamento na definição da base de dados com colunas removidas, já que há uma versão da base que precisa ter dados excluídos por completo, em um processo manual que utiliza o método drop, pré-definido da biblioteca pandas. 
Além disso, também há tratamento na definição da base que contém a esparsidade de dados, onde uma função foi criada para fazer o tratamento de forma aleatória, atribuindo valores que representam a média das colunas no lugar dos valores vindos da simulação

## Experimento

Para o experimento, a base de dados é separada em três níveis:

Base de dados completa: é a base original porém já tratada.

Base de dados esparsa: é esparsada de acordo com a função abaixo, que recebe uma porcentagem e substitui os valores aleatoriamente pela média da coluna que este valor faz parte.

def generate_new_sparse(original_df, perc):
        df_sparse = original_df.copy()

        for column in original_df.copy():
                # Sample de perc% dos itens
                items_to_change = df_sparse.sample(frac=perc)

                # O que sobrou sem o sample
                items_left = df_sparse.drop(items_to_change.index)

                # Operacao de esparsamento
                mean = original_df[column].mean()
                items_to_change[column] = items_to_change[column].apply(lambda x: mean)

                # Insere de volta no dataframe
                df_sparse = pd.concat([items_left, items_to_change])
                df_sparse.sort_index(inplace=True)

        return df_sparse

Base de dados com colunas removidas: 10 colunas foram removidas manualmente e sem critério definido para a remoção.

A partir da definição destas bases, são definidas as saídas:

Links_SumOfCars: O número de total veículos num determinado caminho
Path_LoadedTime: A velocidade de locomoção média dos veículos num determinado caminho Path_LoadedSpeed: Tempo de locomoção média dos veículos num determinado caminho

Com isso, é possível rodar, para cada nível da base de dados, o modelo de árvore de decisão e de random forest, buscando os melhores resultados através de GridSearchCV e armazenando os resultados para comparação.

Obs: para obter um resultado mais preciso, o modelo que lida com a base esparsa roda 10x e extrai a média desses valores, o valor 10 foi escolhido de forma arbitrária.

### Decision tree

Foi utilizada a biblioteca scikt-learn com os seguintes parâmetros:
forest__criterion
forest__max_features
forest__n_estimators

### Random Forest

Foi utilizada a biblioteca scikt-learn com os seguintes parâmetros:
forest__criterion
forest__max_features
forest__n_estimators