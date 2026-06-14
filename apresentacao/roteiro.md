# Roteiro da Apresentação
**Título:** Sistema de Recomendação de Filmes com Fatoração de Matrizes e Centralização por Usuários
**Disciplina:** PO-233 Aprendizado de Máquina — Prof.ª Ana Carolina Lorena
**Duração total:** 15 minutos (~13 slides)
**Referência de tempo:** indicado por slide

---

## Slide 1 — Capa (30s)

**Conteúdo do slide:**
- Título do trabalho
- Nomes: Denilson Gustavo de Araújo Soares e Enzo Mitsuo Tokushige Costa
- ITA / PO-233 / Junho de 2026

**Conceitos que você precisa dominar:**
- Saber resumir o trabalho em uma frase se perguntado na abertura: *"treinamos um modelo de fatoração de matrizes com SVD no dataset MovieLens 25M e mostramos que centralizar as notas por usuário antes da decomposição reduz o RMSE em 72%."*

**O que falar:**
> "Bom dia/Boa tarde. Somos [Denilson] e [Enzo], e vamos apresentar nosso trabalho final de PO-233, que trata de um sistema de recomendação de filmes baseado em fatoração de matrizes com centralização por usuário."

---

## Slide 2 — Agenda (20s)

**Conteúdo do slide:**
- Motivação → Dataset → Pré-processamento → SVD e Centralização → Similaridade de Cosseno → Resultados → Limitações

**O que falar:**
> "A apresentação segue esta ordem: começamos com a motivação do problema, depois descrevemos os dados e o pré-processamento, explicamos o modelo — SVD e similaridade de cosseno —, mostramos os resultados e terminamos com limitações e trabalhos futuros."

---

## Slide 3 — Motivação (1min 30s)

**Conteúdo do slide:**
- O problema de recomendação: esparsidade de 99,74%, média de 0,26% dos filmes avaliados por usuário
- O desafio central: 162.541 × 59.047 = 9,6 bilhões de pares, apenas 25 milhões avaliados
- Abordagem: Fatoração de Matrizes (SVD)
- Meta: estimar nota que o usuário daria a filmes nunca assistidos

**Conceitos que você precisa dominar:**
- **Esparsidade** é a fração de entradas não observadas na matriz usuário × filme. 99,74% significa que 99,74% das combinações possíveis nunca receberam uma avaliação. Isso torna qualquer método que dependa de similaridade direta entre usuários muito instável — dois usuários raramente avaliaram os mesmos filmes.
- **Por que SVD e não similaridade direta (user-user CF ou item-item CF)?** Na similaridade direta, você precisa encontrar usuários com histórico parecido. Com 99,74% de esparsidade, dois usuários quaisquer raramente têm filmes em comum para comparar — o denominador da similaridade de cosseno fica perto de zero. A fatoração de matrizes *generaliza*: aprende representações densas que permitem comparar usuários mesmo sem filmes em comum.
- **O que é "filtragem colaborativa"?** É a família de métodos que recomenda com base no comportamento coletivo dos usuários — sem usar metadados sobre os itens (gênero, diretor, etc.). A premissa é: usuários com histórico parecido tendem a gostar das mesmas coisas.

**O que falar:**
> "Plataformas como Netflix e Amazon têm catálogos com dezenas de milhares de títulos. O desafio fundamental é: como recomendar algo relevante para um usuário que avaliou menos de 0,3% do catálogo? No nosso dataset, essa esparsidade chega a 99,74% — de 9,6 bilhões de pares possíveis, só 25 milhões foram avaliados. Nesse cenário, comparar usuários diretamente é instável: dois usuários raramente têm filmes em comum. A solução é a fatoração de matrizes: aprender representações densas que capturam preferências implícitas e permitem generalizar a partir dos dados escassos que existem."

---

## Slide 4 — Dataset — MovieLens 25M (1min)

**Conteúdo do slide:**
- 25.000.095 avaliações, 162.541 usuários, 59.047 filmes, esparsidade 99,74%
- Notas de 0,5 a 5,0 (passo 0,5), média 3,53, mediana 3,50
- `ratings.csv` e `movies.csv` com suas colunas

**Conceitos que você precisa dominar:**
- **MovieLens 25M** é um dataset público do GroupLens Research (Universidade de Minnesota), amplamente usado como benchmark em pesquisa de sistemas de recomendação.
- As notas são em incrementos de 0,5 — isso significa que o problema é de **regressão**, não de classificação binária. Existe uma escala ordinal contínua.
- **Viés de positividade**: mais de 62,5% das notas são ≥ 3,5. Usuários tendem a avaliar mais filmes de que gostaram — isso já é um sinal de que a média do usuário vai ser sistematicamente diferente da média global, o que motiva a centralização por usuário.
- O campo `genres` em `movies.csv` tem múltiplos gêneros separados por `|` — Drama e Comedy dominam o catálogo.

**O que falar:**
> "O dataset é o MovieLens 25M — 25 milhões de avaliações de 162 mil usuários sobre 59 mil filmes. As notas vão de 0,5 a 5,0 em incrementos de 0,5, com média 3,53. Mais de 62% das notas estão acima de 3,5 — há um viés de positividade claro: usuários avaliam mais o que gostaram. Isso tem implicações para o modelo, que voltamos a discutir no slide de centralização."

---

## Slide 5 — Pré-processamento (1min 30s)

**Conteúdo do slide:**
- Pipeline: remoção de duplicatas (0), conversão timestamp→datetime, remoção de filmes sem avaliações (3.376), filtro usuários ≥ 20 e filmes ≥ 50
- Split temporal 80/20: treino até 2016-05-28 (19.715.942 ratings), teste depois (4.928.986)
- Após filtragem: 162.540 usuários, 13.176 filmes, 24.644.928 ratings

**Conceitos que você precisa dominar:**
- **Por que filtrar usuários com < 20 avaliações e filmes com < 50?** O SVD aprende um vetor latente para cada usuário e para cada filme. Se um usuário tem só 3 avaliações, o vetor latente dele será muito ruidoso — vai se ajustar a esses 3 pontos sem capturar preferências reais. O filtro garante representatividade estatística mínima.
- **Split temporal vs. split aleatório**: um split aleatório mistura avaliações futuras e passadas — o modelo pode "ver" que o usuário deu 5 estrelas para um filme em 2018 e usar isso para predizer a avaliação de 2015. Isso é **vazamento de informação** (*data leakage*). O split temporal evita isso ao treinar só com dados passados e testar com dados futuros, simulando o cenário real de produção.
- **13.176 vs. 12.315 filmes**: após a filtragem ficam 13.176 filmes. Mas quando construímos a matriz esparsa **só com o conjunto de treino**, alguns filmes que aparecem apenas no teste são excluídos — sobram 12.315 filmes na matriz de treino. Esse número (12.315) é o que aparece nos slides de SVD e Similaridade.

**O que falar:**
> "O pré-processamento tem quatro etapas. Primeiro, remoção de duplicatas — nenhuma encontrada. Segundo, conversão de timestamp Unix para datetime. Terceiro, remoção dos 3.376 filmes sem nenhuma avaliação. Quarto, filtro de frequência: mantivemos só usuários com pelo menos 20 avaliações e filmes com pelo menos 50 — isso garante que o SVD aprenda vetores latentes com qualidade estatística mínima. Para a divisão treino/teste, usamos split temporal — os 80% de avaliações mais antigas para treino e os 20% mais recentes para teste. Isso simula o cenário real e evita vazamento de informação, que ocorreria se fizéssemos uma divisão aleatória."

---

## Slide 6 — Fatoração de Matrizes com SVD (2min)

**Conteúdo do slide:**
- Decomposição: R ≈ UΣVᵀ (U = perfil usuário, V = perfil filme, Σ = importância dos fatores)
- Predição: r̂ᵤᵢ = μᵤ + uᵤᵀvᵢ
- Parâmetros: k = 50 fatores, variância explicada 18,10%, esparsidade 98,83%
- Centralização: r̃ᵤᵢ = rᵤᵢ − μᵤ (antes do SVD; μᵤ somado de volta na predição)

**Conceitos que você precisa dominar:**
- **O que o SVD faz matematicamente**: decompõe a matriz R (m×n) no produto U (m×k), Σ (k×k diagonal), Vᵀ (k×n). O TruncatedSVD mantém apenas os k maiores valores singulares — a melhor aproximação de posto k no sentido da norma de Frobenius. Na prática: comprime a matriz em duas matrizes menores (perfis latentes de usuários e filmes).
- **O que são os fatores latentes?** São dimensões abstratas aprendidas automaticamente. Um fator pode capturar algo como "preferência por drama europeu vs. ação americana", outro "filmes de arte vs. blockbusters", etc. O modelo não rotula esses fatores — eles emergem dos padrões de avaliação.
- **Por que 18,10% de variância explicada não é alarmante?** Com 99,74% de esparsidade, a matriz tem muito ruído. Os 50 componentes capturam os padrões mais consistentes — o restante da variância é ruído individual que não deve ser memorizado. Aumentar k ajudaria marginalmente mas aumentaria overfitting.
- **O problema dos zeros**: quando construímos a matriz esparsa, filmes não avaliados recebem zero. O SVD não distingue "não avaliou" de "deu nota 0" — então aprende que a maioria das combinações usuário-filme tem nota próxima de zero, distorcendo os vetores latentes. Esse é o motivo do RMSE sem centralização ser de 3,43 — pior que o baseline ingênuo.
- **Por que a centralização resolve?** Ao subtrair μᵤ de cada nota observada de u, os zeros passam a representar "desvio em relação à média desse usuário" (zero = "nota igual à média dele"), e não "nota zero". Isso reduz o viés sistemático. Na predição, somamos μᵤ de volta: r̂ᵤᵢ = μᵤ + uᵤᵀvᵢ.
- **O que é TruncatedSVD vs. SVD clássico?** O SVD clássico exige que a matriz seja densa e é O(mn²). O TruncatedSVD calcula apenas os k maiores valores singulares de forma iterativa (algoritmo ARPACK/LAPACK), sendo viável para matrizes esparsas de milhões de entradas.

**O que falar:**
> "A fatoração de matrizes comprime a matriz esparsa R — usuários por filmes — em duas matrizes densas menores: U com os perfis latentes dos usuários, e V com os perfis latentes dos filmes. Esses fatores latentes não têm rótulo — o modelo aprende automaticamente dimensões como 'preferência por cinema de arte vs. blockbuster', sem que a gente diga isso explicitamente. Usamos 50 fatores, que explicaram 18,1% da variância — o restante é ruído individual que não deve ser memorizado. O problema central do SVD com matrizes de avaliação é que entradas ausentes são tratadas como zero. Como as notas reais vão de 0,5 a 5,0, o modelo aprende que 'não avaliou' é equivalente a 'nota zero', o que distorce completamente os vetores latentes — e vocês vão ver o impacto disso nos resultados. A solução é a centralização por usuário: antes do SVD, subtraímos a média de cada usuário de suas notas observadas. Os zeros passam a representar 'desvio em relação à média' — uma suposição muito mais razoável. Na predição, somamos a média de volta."

---

## Slide 7 — Similaridade de Cosseno entre Itens (1min 30s)

**Conteúdo do slide:**
- `item_factors`: matriz (12.315 × 50) extraída do SVD
- sim(i,j) = vᵢᵀvⱼ / (‖vᵢ‖‖vⱼ‖) ∈ [0,1]
- Pipeline de recomendação: histórico do usuário → 20 vizinhos mais similares por filme → score ponderado pela nota → Top-10
- Exemplo: *A Fish Called Wanda* → 4 filmes Monty Python + *The Commitments*

**Conceitos que você precisa dominar:**
- **Por que usar similaridade de cosseno no espaço latente?** Os vetores latentes de filmes comprimem padrões de co-avaliação. Filmes que tendem a ser avaliados de forma similar pelos mesmos usuários terão vetores latentes próximos. A similaridade de cosseno mede o ângulo entre esses vetores — independente da magnitude, que pode variar por frequência de avaliações.
- **Como o score é calculado?** Para cada filme i no histórico do usuário com nota rᵤᵢ: acumula-se score(j) += sim(i,j) × rᵤᵢ para todos os vizinhos j. Isso pondera a similaridade pela nota — filmes que o usuário amou (nota 5) influenciam mais o score dos vizinhos do que filmes que ele apenas tolerou (nota 2).
- **Por que 20 vizinhos por filme (TOP_K_SIMILAR = 20)?** É um hiperparâmetro de trade-off: poucos vizinhos = recomendações muito parecidas com o histórico (baixa diversidade); muitos vizinhos = perde a especificidade da similaridade local.
- **A semântica emergente**: o exemplo dos filmes Monty Python ilustra que o modelo aprende coerência semântica *sem supervisão* — nunca foi informado que esses filmes são do mesmo universo. Isso emergiu puramente dos padrões de co-avaliação de 162 mil usuários.

**O que falar:**
> "Com os vetores latentes de filmes aprendidos pelo SVD — uma matriz de 12.315 filmes por 50 componentes — calculamos a similaridade de cosseno entre todos os pares. Para recomendar a um usuário: buscamos os filmes que ele avaliou; para cada um, achamos os 20 mais similares no espaço latente; acumulamos scores ponderando pela nota que o usuário deu — filmes com nota 5 influenciam mais; e retornamos os 10 com maior score, excluindo o que ele já assistiu. O exemplo à direita mostra a qualidade semântica do modelo: os filmes mais similares a *A Fish Called Wanda* são quatro filmes do Monty Python e *The Commitments* — todos dentro do mesmo universo de comédia britânica. O modelo aprendeu isso sem saber nada sobre gênero explicitamente."

---

## Slide 8 — Resultados — RMSE e MAE (1min 30s)

**Conteúdo do slide:**
- Sem centralização: RMSE 3,4339 / MAE 3,2834
- Com centralização: RMSE **0,9635** / MAE **0,7177**
- Baseline ingênuo (média global): RMSE ≈ 1,0221
- Impacto: queda de 72% no RMSE com centralização

**Conceitos que você precisa dominar:**
- **RMSE** (*Root Mean Squared Error*): raiz da média dos erros quadráticos. Penaliza erros grandes proporcionalmente mais que erros pequenos. RMSE = 0,96 significa que, em média, o modelo erra cerca de 0,96 estrelas por predição.
- **MAE** (*Mean Absolute Error*): média dos valores absolutos dos erros. Mais intuitivo: MAE = 0,72 significa que o modelo erra 0,72 estrelas em média, sem penalizar erros grandes extra.
- **Baseline ingênuo**: predizer sempre a média global de treino (~3,53) para qualquer usuário e qualquer filme. O RMSE do baseline ingênuo é aproximadamente igual ao desvio padrão das notas (~1,02). Um modelo que não bate esse baseline é inútil — está fazendo pior do que ignorar o usuário completamente.
- **Por que sem centralização o RMSE é 3,43?** O modelo aprende que a nota "esperada" para qualquer par usuário-filme é próxima de zero (porque 99% das entradas são zero). Para filmes reais com nota ≥ 3, o erro é sempre pelo menos 3. Isso é o problema dos zeros discutido no slide anterior.
- **Por que com centralização o RMSE cai para 0,96?** Os zeros agora representam "desvio zero em relação à média do usuário". O modelo aprende a distinguir preferências individuais — quem gosta mais de um gênero, quem é mais crítico — e prediz com mais precisão.
- **Clipping**: as predições foram limitadas ao intervalo [0,5; 5,0] para evitar valores impossíveis como notas negativas.

**O que falar:**
> "Os resultados mostram claramente o impacto da centralização. Sem ela, o RMSE é de 3,43 — pior do que o baseline ingênuo de sempre predizer a média global, que tem RMSE de 1,02. Isso confirma o problema dos zeros: o modelo aprende que quase todo par usuário-filme tem nota próxima de zero, distorcendo completamente as predições. Com a centralização por usuário, o RMSE cai para 0,96 e o MAE para 0,72 — uma redução de 72% com uma mudança de pré-processamento. Notavelmente, o modelo supera o baseline ingênuo: 0,96 < 1,02. Isso significa que o SVD com centralização captura padrões individuais de preferência além da tendência central."

---

## Slide 9 — Resultados — Precision@10 (1min)

**Conteúdo do slide:**
- Precision@k = |top-k ∩ relevantes| / k
- Candidatos: filmes avaliados no teste e conhecidos pelo modelo
- Relevante: nota ≥ 4,0 — k = 10, avaliado em 200 usuários
- Resultado: **Precision@10 = 0,5743**

**Conceitos que você precisa dominar:**
- **Precision@k** mede a qualidade do ranking: dos k filmes que o modelo recomenda, quantos são de fato relevantes para o usuário? É uma métrica de **ranking**, não de predição de nota — avalia se o modelo coloca os bons filmes no topo da lista.
- **Limiar de relevância θ = 4,0**: consideramos que o usuário "gostaria" de um filme se sua nota real é ≥ 4,0. Abaixo disso, o filme não é relevante para recomendação.
- **Por que restringir os candidatos aos filmes avaliados no teste?** Se os candidatos fossem todos os 12.315 filmes do catálogo, a maioria seria irrelevante por definição (o usuário nunca os viu, então não temos nota real para comparar). Isso inflaria o denominador artificialmente. Ao restringir aos filmes que o usuário efetivamente avaliou no período de teste, a métrica mede a capacidade do modelo de **ranquear** itens cuja relevância é conhecida.
- **Interpretação**: Precision@10 = 0,57 significa que, em média, 5,7 dos 10 filmes recomendados são considerados bons pelo usuário segundo suas avaliações reais no teste. O modelo nunca viu essas avaliações durante o treino.
- **Limitação desta métrica**: por restringir candidatos ao teste, a Precision@10 tende a ser otimista comparada com avaliação sobre o catálogo completo. Isso é reconhecido nas limitações.

**O que falar:**
> "Além do erro de predição de nota, avaliamos a qualidade do ranking com a Precision@10. A métrica pergunta: dos 10 filmes que o modelo recomenda, quantos são de fato relevantes — com nota ≥ 4,0 — segundo as avaliações reais do usuário no período de teste? Uma observação metodológica: os candidatos são restritos aos filmes que o usuário avaliou no teste, não ao catálogo inteiro. Isso evita o problema de inflar o denominador com filmes que nenhum usuário avaliou. O resultado foi 0,57 — em média, 5,7 dos 10 filmes recomendados são considerados bons pelo usuário, sem que o modelo tenha visto essas avaliações no treino."

---

## Slide 10 — Exemplo de Recomendação (1min)

**Conteúdo do slide:**
- Usuário 1: avaliou com nota 5,0 — *Lost in Translation*, *Requiem for a Dream*, *Three Colors: Blue*, *The Seventh Seal*, *Look at Me*
- Top-5 recomendações: *Discreet Charm of the Bourgeoisie* (1972), *Vivre sa vie* (1962), *Andrei Rublev* (1969), *Viridiana* (1961), *Last Year at Marienbad* (1961)

**Conceitos que você precisa dominar:**
- O histórico do usuário 1 é coerente: são filmes de cinema de arte europeu e americano, dramáticos, autorais, de diretores como Sofia Coppola, Darren Aronofsky, Krzysztof Kieślowski, Ingmar Bergman.
- As recomendações geradas — Buñuel, Godard, Tarkovsky, Alain Resnais — são clássicos do cinema de arte europeu do século XX, semanticamente coerentes com o perfil.
- **O ponto central**: o modelo nunca recebeu informação de gênero, diretor ou país de origem. A coerência emergiu puramente dos padrões de co-avaliação — usuários que gostam de *The Seventh Seal* tendem a dar notas altas para *Andrei Rublev*. Isso é a fatoração de matrizes funcionando como esperado.

**O que falar:**
> "Para ilustrar concretamente, o usuário 1 deu nota 5 para filmes como *Lost in Translation*, *Requiem for a Dream* e *O Sétimo Selo* — um perfil claramente voltado para cinema autoral dramático. Os 5 primeiros filmes recomendados são clássicos europeus: *O Discreto Charme da Burguesia* de Buñuel, *Vivre sa Vie* de Godard, *Andrei Rublev* de Tarkovsky. O modelo inferiu preferência por cinema de arte europeu sem receber nenhuma informação explícita de gênero ou diretor — apenas pelos padrões latentes dos 162 mil usuários."

---

## Slide 11 — Limitações e Trabalhos Futuros (1min)

**Conteúdo do slide:**
- **Cold start**: usuários/filmes novos sem vetor latente
- **SVD estático**: não atualiza incrementalmente
- **18,10% de variância explicada**: 50 componentes podem ser insuficientes
- **Precision@10 restrita ao teste**: tende a ser otimista
- Extensões: ALS, features de conteúdo, leave-one-out, mais componentes

**Conceitos que você precisa dominar:**
- **Cold start**: é o problema mais clássico de filtragem colaborativa. Um usuário novo sem histórico não tem vetor latente — o sistema não tem como recomendar nada personalizado. Mesmo após o usuário fazer algumas avaliações, o vetor latente não é atualizado sem retreinar o modelo.
- **Por que ALS é melhor teoricamente?** O ALS (*Alternating Least Squares*) otimiza a função de perda **exclusivamente sobre as entradas observadas** — ignora as entradas ausentes em vez de tratá-las como zero. Isso elimina o viés sistemático dos zeros sem precisar de centralização. A centralização é uma solução parcial eficiente; o ALS é a solução mais correta.
- **Por que aumentar k?** 50 componentes capturam os padrões mais fortes. Com mais componentes, o modelo capturaria preferências mais específicas (subgêneros, diretores específicos, etc.), potencialmente melhorando a qualidade. O trade-off é overfitting e custo computacional.
- **Features de conteúdo (abordagem híbrida)**: usar gênero, ano, tags além das avaliações permitiria recomendar filmes novos (sem avaliações) com base em sua semelhança com itens que o usuário gostou — mitigando cold start de itens.

**O que falar:**
> "O modelo tem limitações importantes a reconhecer. A principal é o cold start: usuários ou filmes novos não têm vetor latente e não podem ser recomendados — é inerente à filtragem colaborativa pura. O SVD também é estático: não atualiza com novas avaliações sem retreinar tudo. Os 18% de variância explicada sugerem que 50 componentes podem ser insuficientes. E a Precision@10 com candidatos restritos ao teste tende a ser otimista. Como próximos passos: substituir SVD por ALS, que otimiza só sobre entradas observadas sem o problema dos zeros; incorporar features de conteúdo para mitigar cold start; e avaliar com o catálogo completo como candidatos para uma métrica mais realista."

---

## Slide 12 — Encerramento (20s)

**Conteúdo do slide:**
- Nomes dos autores
- "Obrigado — Perguntas?"

**Perguntas prováveis e como responder:**

- *"Por que SVD e não ALS ou redes neurais?"* → SVD é a base mais clássica de fatoração de matrizes e o objetivo foi demonstrar o impacto da centralização. ALS seria a evolução natural — e está listado como trabalho futuro. Redes neurais (ex.: NCF) seriam mais capazes mas menos interpretáveis e mais caras computacionalmente.

- *"Por que 50 componentes?"* → É um valor padrão na literatura para datasets de escala similar. Testamos e 18,1% de variância explicada indica que é suficiente para os padrões principais sem overfit. Aumentar k melhoraria marginalmente mas com custo quadrático na similaridade de cosseno.

- *"A Precision@10 de 0,57 é boa?"* → Depende do benchmark. Com candidatos restritos ao teste é uma métrica otimista. O valor indica que o modelo tem qualidade real — 57% de precisão é significativamente melhor que aleatório (que seria ~θ, a proporção de notas ≥ 4 no teste).

- *"Como a centralização resolve o problema dos zeros matematicamente?"* → Antes da centralização, uma entrada ausente e uma nota zero são indistinguíveis para o SVD. Após centralizar por μᵤ, uma entrada ausente (zero na matriz centralizada) representa "desvio zero em relação à média do usuário" — note que os zeros ausentes e observados ainda são misturados, mas agora a média das entradas observadas é zero, então os zeros ausentes são uma interpolação razoável.

**O que falar:**
> "Em resumo: o resultado mais expressivo é que a centralização por usuário — uma mudança simples de pré-processamento — reduziu o RMSE em 72% e é o fator mais crítico para o desempenho do TruncatedSVD em recomendação. O sistema consegue capturar preferências latentes sem nenhuma informação explícita de conteúdo. Estamos à disposição para perguntas."
