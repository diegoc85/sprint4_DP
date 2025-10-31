# Relatório do Projeto: Otimização de Consumo Diário de Insumos na DASA
---

## 1. Introdução ao Problema

Este projeto aborda o desafio de controle de estoque nas unidades de diagnóstico da DASA em que o problema central é a falta de registro preciso do consumo diário de insumos (reagentes e descartáveis), o que leva a dificuldades na previsão de reposição e gera desperdício de material.

O objetivo deste código é aplicar a técnica de Programação Dinâmica (PD) para modelar uma solução, a qual foca em determinar a **quantidade mínima de pacotes de insumos** (ex: caixas de reagentes) que precisam ser abertos para suprir uma demanda diária específica. Ao se achar o número mínimo, minimizamos o desperdício (evitando abrir um pacote grande quando pacotes menores seriam suficientes) e melhoramos a visibilidade do consumo real.

Conforme solicitado na documentação, o problema foi implementado em duas versões que devem, obrigatoriamente, produzir o mesmo resultado:
1.  Recursiva com Memorização (Abordagem *Top-Down*)
2.  Iterativa (Abordagem *Bottom-Up*)

## 2. Abordagem e Estrutura dos Algoritmos Utilizada no Problema

A abordagem de Programação Dinâmica, conforme solicitada na documentação da Sprint 4, foi aplicada para resolver a questão central do consumo. A lógica foca em responder a seguinte pergunta: "Para um determinado consumo_total_necessario (ex: 78 unidades), qual é a menor quantidade de pacotes (tamanhos_pacotes_reagentes) que se deve abrir para somar exatamente esse valor sem faltas ou sobras?"

* **Entrada Principal:**
    1.  `tamanhos_pacotes_reagentes`: Uma lista com os tamanhos dos pacotes disponíveis (ex: [1, 5, 10, 25] unidades).
    2.  `consumo_total_necessario`: O número total de unidades de um insumo que precisamos para a demanda do dia.
* **Saída:**
    1.  O número mínimo de pacotes que devem ser usados para atingir *exatamente* o consumo necessário.
    2.  Um detalhamento de *quais* pacotes e *quantos* de cada foram usados para compor a melhor solução.
    3.  Se for impossível atingir o valor exato, o algoritmo irá retornar o seguinte texto: "Impossivel suprir essa demanda!".

---

### Versão 1: Recursiva com Memorização (Top-Down)

Esta abordagem resolve o problema "de cima para baixo". A função principal, `calcular_minimo_pacotes_recursivo`, tenta resolver o problema maior (o `consumo_total_necessario`) quebrando-o em problemas menores.

**Estruturas Utilizadas:**

* **Função Principal (`calcular_minimo_pacotes_recursivo`):**
    * Esta é a função "de entrada" que o usuário chama. Sua responsabilidade principal é inicializar o `cache_calculo_novo` (um dicionário do Python).
    * Seguindo a dica dada pelo professor em seu material de aula de limpar o cache, ao iniciar um novo dicionário vazio a cada chamada, garante-se que ele seja "limpo" e que resultados de cálculos antigos não interfiram em novas simulações.
    * Ela então chama a função auxiliar (`_resolver_consumo_recursivo`) para iniciar o processo.

* **Função Auxiliar (`_resolver_consumo_recursivo`):**
    * Esta é a função recursiva que contém a lógica principal do código. Ela tenta "subtrair" cada `tamanho` de pacote do `consumo_restante`.
    * **Caso Base 1:** Se `consumo_restante` for 0, encontramos uma solução (0 pacotes necessários).
    * **Caso Base 2:** Se `consumo_restante` for negativo, significa que o pacote usado foi grande demais (solução inválida, retornamos `float('inf')`).

* **Memorização (`cache_calculo_novo`):**
    * Este é um dicionário usado para armazenar os resultados dos subproblemas já resolvidos (memorização).
    * Antes de calcular o mínimo de pacotes para um `consumo_restante`, verificamos se ele já está no `cache_calculo_novo`. Se estiver, retornamos o valor salvo imediatamente.
    * Isso evita o re-cálculo exponencial dos mesmos subproblemas tornando a solução viável e mais ágil. 
    * **Detalhamento**: Para permitir o detalhamento dos pacotes, este cache armazena uma *tupla* com duas informações: `(contagem_minima, melhor_pacote_usado)`.

* **Ajuste do limite da Recursão (`sys.setrecursionlimit`):**
    * Para garantir que a função recursiva suporte entradas grandes (como uma demanda de 1000 ou 2000 unidades), o limite de recursão padrão do Python foi aumentado no início do código. Sem isso, o programa falharia com um `RecursionError` para demandas maiores. Essa também foi uma dica dada pelo professor em seu material de aula. 

* **Reconstrução do Caminho (`pacotes_selecionados`):**
    * Após a função principal encontrar a contagem mínima (ex: 6), o `while` é executado.
    * Este laço funciona de trás para frente, isto é, ele começa com o consumo total (ex: 78) e olha no `cache_calculo_novo` qual foi o `melhor_pacote_usado` salvo para aquele valor (ex: pacote 25).
    * Ele adiciona o pacote 25 ao dicionário `pacotes_selecionados` e subtrai seu valor (78 - 25 = 53).
    * Ele repete o processo, agora para 53, olhando no cache qual foi o melhor pacote salvo para 53 e assim por diante, até o consumo chegar a 0. Isso gera o detalhamento de quais pacotes formaram a melhor solução.

### Versão 2: Iterativa (Bottom-Up)

Esta abordagem resolve o problema de baixo para cima e em vez de começar do consumo total e descer, ela começa do consumo 0 e constrói a solução passo a passo até atingir o `consumo_total_necessario`.

**Estruturas Utilizadas:**

* **Função (`calcular_minimo_pacotes_iterativo`):**
    * Contém toda a lógica de forma linear, sem recursão.

* **Tabela de Programação Dinâmica (`pacotes_minimos_para_consumo`):**
    * Esta é a estrutura central. É uma lista (array) de tamanho `consumo_total_necessario + 1`.
    * O índice `i` desta lista armazena o número mínimo de pacotes necessários para atingir um consumo de *exatamente* `i` unidades.
    * `pacotes_minimos_para_consumo[0]` é inicializado com 0 (caso base: 0 consumo = 0 pacotes).
    * Todos os outros valores são inicializados com `float('inf')` (representando ainda não calculado ou impossível de atingir). O float('inf') é a forma do Python de representar o conceito matemático de infinito positivo. Foi usado o float('inf') como um valor inicial "impossível" para procura o menor (ou mínimo) valor de alguma coisa. É um valor especial do tipo "float" (número decimal) que tem uma propriedade muito útil: ele é maior do que qualquer outro número.

* **Tabela de Rastreamento (`ultimo_pacote_usado`):**
    * Para permitir o detalhamento de quais pacotes foram usados, uma segunda tabela (lista) foi criada.
    * O índice `i` desta lista armazena *qual foi o último pacote usado* (ex: 10) para se obter o melhor resultado guardado em `pacotes_minimos_para_consumo[i]`.
    * Toda vez que o algoritmo encontra uma solução melhor (uma `contagem_prop` menor), ele atualiza *ambas* as tabelas: a `pacotes_minimos_para_consumo` (com a nova contagem) e a `ultimo_pacote_usado` (com o `tamanho` do pacote que permite isso acontecer).

* **Loop Principal (Iteração `consumo_atual`):**
    * O primeiro loop itera de 1 até `consumo_total_necessario`. Ele é responsável por preencher cada célula da `pacotes_minimos_para_consumo`, resolvendo um subproblema de cada vez, em ordem crescente.

* **Loop Aninhado (Iteração `tamanho`):**
    * Para cada `consumo_atual` que estamos tentando resolver, este loop testa todos os `tamanhos_pacotes_reagentes` disponíveis.
    * A lógica principal é: O mínimo de pacotes para o `consumo_atual` é o valor mínimo entre o valor que já está lá e (1 + `pacotes_minimos_para_consumo[consumo_atual - tamanho]`).
    * O algoritmo está sempre fazendo uma comparação para otimizar o resultado.

* **Reconstrução do Caminho (`pacotes_selecionados`):**
    * Assim como na versão recursiva, após a tabela ser totalmente preenchida, um laço `while` é executado.
    * Ele começa no consumo total (ex: 78) e usa a tabela `ultimo_pacote_usado` como um "mapa" para voltar ao início.
    * Ele verifica `ultimo_pacote_usado[78]` (pacote 25), adiciona o 25 ao dicionário `pacotes_selecionados`, e "pula" para o consumo anterior (78 - 25 = 53).
    * Ele então verifica `ultimo_pacote_usado[53]` e repete o processo até o consumo chegar a 0 montando o detalhamento completo.

## 3. Verificação de Resultados

No bloco de execução, são executados três cenários de teste distintos. Ao final de cada cenário, os resultados das versões recursiva e iterativa são comparados com uma declaração `if`.

Esta verificação é importante para garantir que ambas as implementações, apesar de terem lógicas de execução opostas (Top-Down vs. Bottom-Up), chegam exatamente à mesma solução ótima, validando, assim, ambas abordagens conforme solicitado na documentação da Sprint 4.