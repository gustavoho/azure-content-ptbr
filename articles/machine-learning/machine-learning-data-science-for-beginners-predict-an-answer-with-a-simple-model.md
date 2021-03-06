<properties
   pageTitle="Prever uma resposta com um modelo simples - Ciência de dados para iniciantes | Microsoft Azure"
   description="Como criar um modelo simples para prever o preço de um diamante em Ciência de dados para iniciantes, vídeo 4. Inclui uma regressão linear básica com dados de destino."                                  
   keywords="criar um modelo, modelo simples, modelo de dados simples, previsão de preço, modelo de regressão simples"
   services="machine-learning"
   documentationCenter="na"
   authors="brohrer-ms"
   manager="paulettm"
   editor="cjgronlund"/>

<tags
   ms.service="machine-learning"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="06/29/2016"
   ms.author="cgronlun;brohrer;garye"/>

# Ciência de Dados para Iniciantes - vídeo 4: Prever uma resposta com um modelo simples

Saiba como criar um modelo simples para prever o preço de um diamante em Ciência de dados para iniciantes, vídeo 4. Vamos desenhar um modelo de regressão com dados de destino.

Para aproveitar ao máximo a série, assista aos vídeos na ordem. [Acesse a lista de vídeos](#other-videos-in-this-series)

> [AZURE.VIDEO data-science-for-beginners-series-predict-an-answer-with-a-simple-model]

## Transcrição: Preveja uma resposta com um modelo simples

Bem-vindos ao quarto vídeo da série "Ciência de dados para iniciantes". Neste vídeo, vamos criar um modelo simples e fazer uma previsão.

Um *modelo* é uma história simplificada sobre nossos dados. Mostrarei o que quero dizer.

## Colete dados relevantes, precisos, conectados e suficientes

Vamos supor que eu queira comprar um diamante. Eu tenho um anel que pertencia a minha avó com um espaço para um diamante de 1,35 quilate, e eu quero ter uma ideia de quanto esse diamante custará. Levo um bloco de notas e uma caneta até a loja de joias e anoto o preço e o quilate de todos os diamantes em exposição. Começando com o primeiro diamante: ele tem 1,01 quilate e custa $7.366.

Agora eu faço isso para os outros diamantes na loja.

![Colunas de dados de diamante](./media/machine-learning-data-science-for-beginners-predict-an-answer-with-a-simple-model/diamond-data.png)

Observe que a nossa lista tem duas colunas. Cada coluna tem um atributo diferente - peso em quilate e preço - e cada linha é um único ponto de dados que representa um único diamante.

Na verdade, criamos um pequeno conjunto de dados aqui; uma tabela. Observe que isso atende aos nossos critérios de qualidade:

* Os dados são **relevantes**; o peso está definitivamente relacionado ao preço
* Eles são **precisos**; verificamos os preços que anotamos mais de uma vez
* Ele são **conectados**; não há espaço em branco em qualquer uma dessas colunas
* E, como veremos, eles têm uma quantidade **suficiente** de dados para responder à nossa pergunta

## Faça uma pergunta inteligente

Agora, faremos nosso pergunta de uma forma direta: "Quanto custará para comprar um diamante de 1,35 quilate?"

Nossa lista não tem um diamante de 1,35 quilate, então teremos que usar o restante de dados para obter uma resposta para a pergunta.

## Plotar os dados existentes

A primeira coisa que faremos é desenhar uma linha horizontal de números, chamada de eixo, para colocar os pesos no gráfico. A faixa dos pesos vai de 0 a 2, portanto, vamos desenhar uma linha que cubra essa faixa e colocar marcações a cada meio quilate.

Em seguida, desenharemos um eixo vertical para registrar o preço e o conectaremos ao eixo horizontal de peso. Isso será em unidades de dólares. Agora temos um conjunto de eixos coordenados.

![Eixos de peso e preço](./media/machine-learning-data-science-for-beginners-predict-an-answer-with-a-simple-model/weight-and-price-axes.png)

Agora, vamos pegar esses dados e transformá-los em uma *dispersão*. Essa é uma excelente maneira de visualizar conjuntos de dados numéricos.

Para o primeiro ponto de dados, desenhamos uma linha vertical no quilate 1,01. Em seguida, desenhamos uma linha horizontal em $7.366. No local onde elas se encontram, desenhamos um ponto. Isso representa nosso primeiro diamante.

Agora fazemos isso para cada diamante nesta lista. Quando terminarmos, este será o resultado: um monte de pontos, um para cada diamante.

![Plotagem de dispersão](./media/machine-learning-data-science-for-beginners-predict-an-answer-with-a-simple-model/scatter-plot.png)

## Desenhar o modelo usando os pontos de dados

Agora, se você os pontos e semicerrar os olhos, a coleção parecerá uma linha espessa e difusa. Podemos usar nosso marcador para desenhar uma linha reta através deles.

Desenhando uma linha, criamos um *modelo*. Pense nisso como pegar o mundo real e fazer uma versão simples em desenho dele. Agora, o desenho está incorreto, a linha não passa por todos os pontos de dados. Mas, é uma simplificação útil.

![Linha da regressão linear](./media/machine-learning-data-science-for-beginners-predict-an-answer-with-a-simple-model/linear-regression-line.png)

O fato de que todos os pontos não passam exatamente pela linha não tem qualquer problema. Cientistas de dados explicam isso dizendo que há o modelo (essa é a linha) e cada ponto tem algum *ruído* ou *variação* associada a ele. Há a relação perfeita subjacente, e há o mundo real que adiciona ruído e incerteza.

Como estamos tentando responder à pergunta *quanto custa?*, isso é chamado de uma *regressão*. E, como estamos usando uma linha reta, é uma *regressão linear*.

## Usar o modelo para encontrar a resposta

Agora, temos um modelo e fazemos a nossa pergunta: quanto custará um diamante de 1,35 quilate?

Para responder à nossa pergunta, nós identificamos visualmente o 1,35 quilate e desenhamos uma linha vertical. Onde ela cruzar a linha do modelo, identificamos visualmente uma linha horizontal no eixo de dólar. Ela atinge diretamente 10.000. Pronto! Essa é a resposta: um diamante de 1,35 quilate custa aproximadamente $10.000.

![Encontrar a resposta no modelo](./media/machine-learning-data-science-for-beginners-predict-an-answer-with-a-simple-model/find-the-answer.png)

## Criar um intervalo de confiança

É natural se preocupar com a precisão dessa previsão. É muito útil saber se o preço do diamante de 1,35 quilate será muito próximo de $10.000, mais barato ou mais caro. Para descobrir isso, vamos desenhar um envelope ao redor da linha de regressão que inclua a maioria dos pontos. Esse envelope é chamado de nosso *intervalo de confiança*: estamos bem confiantes de que os preços se enquadram nesse envelope, pois, no passado, a maioria deles se enquadrou. Podemos desenhar outras duas linhas horizontais a partir das quais a linha de 1,35 quilate cruza a parte superior e inferior do envelope.

![Intervalo de confiança](./media/machine-learning-data-science-for-beginners-predict-an-answer-with-a-simple-model/confidence-interval.png)

Agora podemos dizer algo sobre o intervalo de confiança: podemos dizer com segurança que o preço de um diamante de 1,35 quilate é de aproximadamente $10.000, mas pode ser $8.000 e pode ser $12.000.

## Pronto, sem matemática ou computadores

Fizemos o que os cientistas de dados são pagos para fazer, e fizemos isso apenas desenhando:

* Fizemos uma pergunta que pudemos responder com dados
* Criamos um *modelo* usando a *regressão linear*
* Fizemos uma *previsão*, completa com um *intervalo de confiança*

E não usamos matemática ou computadores para isso.

Agora, se tivéssemos mais informações, como...

* o design do diamante
* variações de cor (quão próximo do branco o diamante é)
* o número de inclusões no diamante

…teríamos mais colunas. Nesse caso, a matemática se torna útil. Se você tiver mais de duas colunas, é difícil desenhar pontos no papel. A matemática permite o encaixe perfeito dessa linha ou desse plano aos seus dados.

Além disso, se em vez de apenas um punhado de diamantes, tivéssemos dois mil ou dois milhões, esse trabalho seria muito mais otimizado com um computador.

Hoje, falamos sobre como fazer a regressão linear e fizemos uma previsão usando dados.

Confira outros vídeos da série "Ciência de dados para iniciantes" no Aprendizado de Máquina do Microsoft Azure.


## Outros vídeos nesta série

*Ciência de dados para iniciantes* é uma breve introdução à ciência de dados em cinco vídeos curtos.

  * Vídeo 1: [As cinco perguntas que a ciência de dados responde](machine-learning-data-science-for-beginners-the-5-questions-data-science-answers.md)
  * Vídeo 2: [Seus dados estão prontos para a ciência de dados?](machine-learning-data-science-for-beginners-is-your-data-ready-for-data-science.md)
  * Video 3: [Faça uma pergunta que você possa responder com dados](machine-learning-data-science-for-beginners-ask-a-question-you-can-answer-with-data.md)
  * Vídeo 4: Preveja uma resposta com um modelo simples
  * Vídeo 5: [Copie o trabalho de outras pessoas para fazer a ciência de dados](machine-learning-data-science-for-beginners-copy-other-peoples-work-to-do-data-science.md)

## Próximas etapas

  * [Tenha sua primeira experiência da ciência de dados com o Aprendizado de Máquina do Azure](machine-learning-create-experiment.md)
  * [Obtenha uma introdução ao Aprendizado de Máquina no Microsoft Azure](machine-learning-what-is-machine-learning.md)

<!---HONumber=AcomDC_0706_2016-->