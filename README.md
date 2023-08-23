# Previsão consumo medicamento

# Introdução 
A programação de medicamentos é uma atividade chave no fluxo da Assistência Farmacêutica no Sistema Único de Saúde (SUS), sendo aplicada para estimar as quantidades a serem adquiridas e distribuídas pelos gestores municipais, estaduais e federais. Uma programação inadequada pode refletir no abastecimento, e consequentemente na falta de medicamento nas farmácias do SUS, bem como no desperdício de dinheiro publico quando ocorre uma superestimativa nos quantitativos.
Contudo, estimar o consumo de dezenas de medicamentos no SUS muitas vezes não é uma tarefa fácil e rápida, pois envolve a avaliação de vários critérios. Dessa forma, a aplicação de um modelo de predição para series temporais pode ser útil para aumentar a assertividade das predições, além de reduzir o esforço dos profissionais que atuam na assistência farmacêutica.
Uma serie temporal é uma sequência de observações realizadas em um período. A análise de series temporais permite descrever a dependência dos dados em relação ao tempo e prever valores futuros. Este padrão pode ser do tipo tendencialidade, sazonalidade, ciclicidade ou aleatoriedade.  No caso da Assistência Farmacêutica, é possível utilizar os dados de quantidade dispensada para criar uma serie temporal e ser aplicada em um modelo de programação de medicamentos.

# Objetivo
Testar o algoritmo ARIMA (Média Móvel Integrada Autorregressiva) para a predição de consumo em âmbito nacional dos medicamentos filgrastim 300mcg injetável, golimumabe 50mg solução injetável e imiglucerase 400U injetável, pertencentes ao elenco do Componente Especializado da Assistência Farmacêutica (CEAF), a partir dos registros de dispensação do Sistema de Informações Ambulatoriais do SUS (SIA/SUS). 

# Aspectos metodológicos
## Coleta dos dados
A coleta de dados foi realizada em 20 de novembro de 2022 e foram utilizados os registros de APAC provenientes do sistema SIA/SUS para a criação da série temporal de dispensação. Foram selecionados os dados entre janeiro de 2019 a dezembro de 2021, abrangendo os dados das 27 unidades da federação
Para a obtenção dos dados do Sistema de Informações Ambulatoriais do SUS (SIA/SUS) foi utilizada um programa disponibilizado no GitHub (https://github.com/ricardoronsoni/importar-apac-ceaf) que realiza o download dos dados do FTP do Datasus e persiste os mesmos em um banco de dados relacional.  
Foram obtidos 94.254.375 registros de dispensação. 
## Seleção dos medicamentos
Foram selecionados três medicamentos disponíveis no Sistema Único de Saúde (SUS) para a realização desse estudo. Os seguintes critérios de inclusão foram utilizados:
1-	Estar no elenco do Componente Especializado da Assistência Farmacêutica (CEAF), pois apenas os medicamentos pertencentes a este Componente possuem dados abertos;  
1-	Possuir no mínimo 20 meses de série histórica;  
2-	Ser medicamento de compra centralizada, visando diminuir a interferência do desabastecimento nos registros de dispensação.  
Adicionalmente, foram selecionados medicamentos de grupos terapêuticos distintos e para tratamento de situações clínicas distintas, visando aumentar a aleatoriedade das amostras.  
Dessa forma, foram selecionados os seguintes medicamentos:  
1-	Filgrastim 300mcg injetável;  
2-	Golimumabe 50mg solução injetável;  
3-	Imiglucerase 400U injetável.  

## Preparação dos dados
O SIA/SUS disponibiliza os seus dados de forma individualizada, sendo que para obter os dados para o presente estudo foi preciso somar as quantidades dispensadas de cada medicamento e agrupá-los por competência, formando assim uma série histórica de dispensação.  
Adicionalmente, o SIA/SUS identifica os medicamentos do CEAF pelo código de procedimento do Sistema de Gerenciamento da Tabela de Procedimentos, Medicamentos e OPM do SUS (SIGTAP). Sendo assim, foi necessário acessar o sítio eletrônico do SIGTAP (http://sigtap.datasus.gov.br/tabela-unificada/app/sec/inicio.jsp) para proceder com a pesquisa dos códigos de procedimento para cada medicamento selecionado. Essa pesquisa foi realizada em 23 de novembro de 2022.  
Foram localizados os seguintes códigos de procedimento:  
| Medicamento                       | Procedimento |
|-----------------------------------|--------------|
| Filgrastim 300mcg injetável       | 0604250010   |
| Golimumabe 50mg solução injetável | 0604380089   |
| Imiglucerase 400U injetável           | 0604240031   | 
Os dados foram exportados do banco de dados por meio do seguinte SQL (Structured Query Language), onde a variável `$1` corresponde ao código de procedimento de cada medicamento. Foi gerado um arquivo CSV para cada medicamento.  
```
select 
	m.competencia_dispensacao competencia, 
	sum(m.quantidade_aprovada) qtd_dispensada
from apac.medicamento m 
where 
	m.competencia_dispensacao between to_date('2019-01-01', 'YYYY-MM-DD') and to_date('2021-12-31', 'YYYY-MM-DD')
	and m.procedimento = $1
group by 
	m.competencia_dispensacao
order by 1;
```
## Calibração do modelo
Por se tratar de uma série temporal de três anos, o dataset foi constituído por 36 termos, sendo que o mesmo foi dividido em conjunto de treinamento e teste.
Foi aplicada a proporção 80/20 para divisão da série, onde 28 meses foram utilizados para compor o dataset de treinamento e 8 meses para o de testes. Os dados de testes foram sempre os últimos 8 meses das séries.   
  
As demais informações acerca da metodologia empregada, resultados e discussão estão descritos ao longo do script Python, a fim de tornar o texto mais relacionado com os dados obtidos.
