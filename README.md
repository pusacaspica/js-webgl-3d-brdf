# js-webgl-3d-brdf
 Repositório para o segundo trabalho da matéria de Computação Gráfica do Mestrado Strictu Sensu da Universidade Federal Fluminense

 Este trabalho executa uma versão dos BRDFs Cook-Torrance e Ward na forma de shaders sobre modelos fornecidos no formato .json, dentro de uma cena com controles simples para manipular a posição do ponto de luz e a rotação e translação do modelo na cena. A seleção do modelo, por ora, deve ser feita alterando a linha 583 do arquivo index.html. Mais instruções sobre o formato do arquivo .json abaixo. Também existem implementações dos shaders Phong, Lambert e Gouraud para fins de comparação.

 O Cook-Torrance funciona somando uma componente difusa a uma componente especular para resultar em cor em um determinado pixel, dados os parâmetros apropriados, a iluminação na cena e as cores tanto do material difuso quanto do especular. A definição do valor de difusão é um valor escalar que varia de 0.0 até 1.0. Já o especular utiliza algumas funções para simular o comportamento do material sob uma luz específica em condições específicas. A primeira função a ser utilizada é a função de distribuição de microfacetas baseada em Blinn-Phong. 
 
![logo](/common/readme/blinn-phong.png "Função de distribuição de microfacetas")

 Existe implementação no código para a implementação da função de Beckmann entre as linhas 233 e 237, mas ela foi preterida uma vez que era mais fácil fazer depuração da corretude do modelo de Blinn-Phong. Cook-Torrance em sua implementação atual também implementa a própria fórmula de Cook-Torrance para atenuação geométrica.

 Ward, por sua vez, foi implementado em maior parte conforme o artigo original (WARD, 1992). A única diferença foi o uso da aproximação para calcular valores de cossenos e tangentes, que foram aproximadas para melhor desempenho computacional conforme (WALTER, 2005).

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
