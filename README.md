# js-webgl-3d-brdf
 Repositório para o segundo trabalho da matéria de Computação Gráfica do Mestrado Strictu Sensu da Universidade Federal Fluminense

 Este trabalho executa uma versão dos BRDFs Cook-Torrance e Ward na forma de shaders sobre modelos fornecidos no formato .json, dentro de uma cena com controles simples para manipular a posição do ponto de luz e a rotação e translação do modelo na cena. A seleção do modelo, por ora, deve ser feita alterando a linha 583 do arquivo index.html. Mais instruções sobre o formato do arquivo .json abaixo. Também existem implementações dos shaders Phong, Lambert e Gouraud para fins de comparação.

 O Cook-Torrance funciona somando uma componente difusa a uma componente especular para resultar em cor em um determinado pixel, dados os parâmetros apropriados, a iluminação na cena e as cores tanto do material difuso quanto do especular. A definição do valor de difusão é um valor escalar que varia de 0.0 até 1.0. Já o especular utiliza algumas funções para simular o comportamento do material sob uma luz específica em condições específicas. A primeira função a ser utilizada é a função de distribuição de microfacetas baseada em Blinn-Phong, que usa um parâmetro denominado *roughness*.  Existe implementação no código para a implementação da função de Beckmann entre as linhas 233 e 237, mas ela foi preterida uma vez que era mais fácil fazer depuração da corretude do modelo de Blinn-Phong.
 
![logo](/common/readme/blinn-phong.png "Função de distribuição de microfacetas de Blinn-Phong")

Blinn-Phong usa um produto escalar de dois vetores; no caso, o vetor de normal e o *half-vector*; um vetor que é o resultado da soma da direção da luz com o ponto no espaço do observador da cena.

As outras funções usadas são a de atenuação geométrica, que não usa nenhum parâmetro fornecido pelo usuário, e a aproximação de Schlick para cálculo do Fresnel. No caso da atenuação geométrica, este código implementa a própria fórmula de Cook-Torrance para atenuação geométrica.

![logo](/common/readme/geometric_attenuation.png "Função de atenuação geométrica de Cook-Torrance")

Esta função utiliza vários *dot products* entre vários vetores presentes na cena, e basicamente elege o valor mais relevante dentre os três apresentados. Além de usar vetores já vistos na fórmula de Blinn-Phong, são usados também os vetores do observador (i.e. o vetor que emana do observador até o objeto da cena, caracterizado na fórmula como V) e do raio de luz (caracterizado por L).

Por fim, a fórmula da Aproximação de Schlick usa como parâmetro um valor definido pelo usuário, a *reflectância*.

![logo](/common/readme/schlick.png "Aproximação de Schlick")

Essas três fórmulas são somadas para resultar no termo especular em determinada região do modelo. Então, a especularidade final é dada por:

![logo](/common/readme/specular_cook.png "Fórmula da Especularidade")

E a cor final de determinado fragmento do modelo, por sua vez, é dada por:

![logo](/common/readme/cook.png "Fórmula da Especularidade")

As cores difusa e especular são, respectivamente, #FFFFFF e #808080.

 Ward, por sua vez, foi implementado em maior parte conforme o artigo original (WARD, 1992). A única diferença foi o uso da aproximação para calcular valores de cossenos e tangentes, que foram aproximadas para melhor desempenho computacional conforme (WALTER, 2005). A cor segue o mesmo método do BRDF de Cook-Torrance, com a soma de uma componente difusa e uma componente especular, sendo a diferença entre elas a forma como a conta das cores são feitas.

 A componente difusa na BRDF de Ward é calculada por um escalar, mas diferente de Cook-Torrance, há a divisão desse escalar pelo valor de Pi. A componente especular, por sua vez, é aproximada através de:

 ![logo](/common/readme/specular_ward.png "Fórmula da Especularidade")

 Aqui, temos um produto escalar entre o *half-vector* previamente estabelecido e vetores X e Y não vistos antes. Neste código, X é definido como o produto vetorial entre a normal e a multiplicação do *half-vector* pelo vetor (1.0, 0.0, 1.0), e Y é o produto vetorial entre a normal e X.

 **Para executar o código**, deve-se instalar um web server para rodar os exemplos de forma local, uma vez que existe dependência entre o arquivo principal (robot.html) e bibliotecas auxiliares providas pela Packt Publishing localizadas na pasta ./utils. Eu recomendo usar um Python Server (https://developer.mozilla.org/en-US/docs/Learn/Common_questions/set_up_a_local_testing_server), uma vez que todos os testes locais foram feitos usando um Python Server.

**O arquivo .json do modelo 3D** deve estar formatado de uma maneira específica para funcionar. Ele deve ser um arquivo .json com três campos contendo informações pertinentes dos modelos 3D representados: um campo de vértices, um campo com informação sobre normais e um campo com informação de adjacência entre vértices. Tanto o código de index.html quanto o arquivo .json podem ser facilmente estendidos para receber mais informações sobre modelos, contudo, como cores, mapeamento UV ou textura.

index.html conta com uma coleção de funções:
* getShader(type, path)
    * Retorna um shader compilado de determinado tipo (vertex ou fragment) localizado em um caminho específico. Esta função foi utilizada em alguma versão anterior, mas na versão atual ela é perfeitamente substituída por funções de uma das bibliotecas utilitárias.
* getModel(path)
    * Retorna um modelo em .json localizado em um caminho específico. Esta função foi utilizada em alguma versão anterior, mas na versão atual ela é perfeitamente substituída por funções de uma das bibliotecas utilitárias.
* initProgram()
    * Prepara a inicialização do programa, atribuindo os shaders apropriados para as variáveis apropriadas; acontece na primeira vez em que a página é carregada.
* initAttr()
    * Inicializa atributos dos shaders, atribuindo valores iniciais atrelados a variáveis no corpo do script em .js
* initControls()
    * Inicializa a GUI, atrelando controladores da GUI para variáveis que controlam atributos de shaders e de objetos na cena
* render()
    * Renderiza a tela fazendo a chamada da função draw()
* load()
    * Carrega modelo 3D na forma de um arquivo .json e as informações sobre normais, vértices e índices em buffers.
    * Carrega modelo 3D em uma pilha de modelos para possível desenho de vários modelos distintos (precisaria de adaptações para tal)
    * Alimenta o VertexArrayObject e o IndexBufferObject
* draw()
    * Atualiza atributos dos shaders e propriedades dos objetos na cena
        * Posição e rotação dos objetos na cena
        * Posição do ponto de luz
        * Propriedades dos shaders conforme o shader utilizado:
            * **Lambert, Phong e Gouraud**: "Shineness", luz ambiente, difusão e especularidade do material
            * **Cook-Torrance**: Difusão, Roughness e Fresnel
            * **Ward**: Difusão, magnitude do lóbulo de brilho anisotrópico, alfa X e alfa Y
    * Desenha conteúdo dos buffers na tela
* init()
    * Inicializa programa na primeira vez que a página é carregada
