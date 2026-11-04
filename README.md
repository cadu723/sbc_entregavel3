# Mini-Projeto 3 — Ontologia de Triagem Médica e Doenças (RDF, RDFS, OWL e SPARQL)

Este repositório contém uma base de conhecimento modelada como grafo de conhecimento (Web Semântica) no domínio médico clínico. O sistema simula um motor de inferência e triagem diagnóstica, inspirado no MYCIN, capaz de cruzar sintomas, doenças, agentes patogênicos, tratamentos e pacientes.

O projeto foi desenvolvido em Python com as bibliotecas `rdflib` e `owlrl`, e implementa consultas programáticas (`g.triples()`), consultas e atualizações declarativas (SPARQL) e uma verificação das construções OWL utilizadas.

> Os valores clínicos da base (letalidade, dosagem, incubação) são ilustrativos, para fins acadêmicos, e não substituem fontes médicas.

---

## 1. Resumo da base e inferência (reasoner OWL-RL)

A base (`ontologia_doencas.ttl`) modela doenças, sintomas, agentes patogênicos, tratamentos, médicos e pacientes. Ao carregar o arquivo e aplicar o reasoner `DeductiveClosure(OWLRL_Semantics)`, o script imprime as seguintes métricas:

| Métrica | Valor |
|---|---|
| Triplas explícitas (carregadas do `.ttl`) | **325** |
| Triplas inferidas pelo reasoner | **692** |
| Total após o fechamento dedutivo | **1017** |
| Inconsistências detectadas na base | **0** |
| Classes | **16** |
| Propriedades | **16** (9 de objeto e 7 de dados) |
| Instâncias (A-Box) | **40** |

As 692 triplas inferidas incluem inferências do domínio (inversas, transitividade, simetria, equivalência de classes, herança de classes, domain e range) e também axiomas padrão do fechamento OWL-RL, como `rdf:type owl:Thing`, `rdfs:Resource` e `owl:sameAs` reflexivo.

---

## 2. Arquivos do projeto

- `ontologia_doencas.ttl`: base de conhecimento em Turtle, com a T-Box (esquema e regras) e a A-Box (instâncias).
- `consultas_doencas.py`: script principal. Carrega a base, aplica o reasoner e executa 17 consultas e atualizações (8 com `g.triples()` e 9 com SPARQL), seguidas de 3 verificações das construções OWL.
- `README.md`: este arquivo.

---

## 3. Estrutura da ontologia (T-Box)

A taxonomia tem 16 classes, organizadas por `rdfs:subClassOf` em duas árvores, com até 3 níveis abaixo da raiz `:EntidadeSaude`.

```text
:EntidadeSaude
 ├── :Doenca
 │    ├── :DoencaInfecciosa
 │    │    ├── :DoencaViral
 │    │    └── :DoencaBacteriana
 │    └── :DoencaCronica
 ├── :Sintoma (owl:equivalentClass :ManifestacaoClinica)
 ├── :ManifestacaoClinica
 ├── :Tratamento
 │    ├── :Medicamento
 │    └── :TerapiaSuporte
 └── :AgentePatogenico

:Pessoa
 ├── :Paciente
 │    └── :PacienteTriagemCritica
 └── :Medico
```

### Propriedades

- **9 propriedades de objeto:** `:apresentaSintoma`, `:sintomaDe`, `:tratadaCom`, `:indicadoPara`, `:causadaPor`, `:evoluiPara`, `:interageCom`, `:diagnosticadoCom`, `:atendidoPor`.
- **7 propriedades de dados:** `:codigoCID`, `:taxaLetalidade`, `:periodoIncubacaoDias`, `:gravidadeSintoma`, `:dosagemMg`, `:idadePaciente`, `:statusClinico`.

Todas têm `rdfs:domain` e `rdfs:range` declarados.

### Construtores OWL utilizados

1. **Propriedades inversas (`owl:inverseOf`):** `:apresentaSintoma` ↔ `:sintomaDe` e `:tratadaCom` ↔ `:indicadoPara`. Declaramos só um sentido, e o reasoner materializa o outro (consultas 1.3 e 1.5 e atualização 2.8).
2. **Propriedade transitiva (`owl:TransitiveProperty`):** `:evoluiPara` (Dengue Clássica → Dengue com Sinais de Alarme → Dengue Grave). O reasoner deduz que a Dengue Clássica também evolui para a Dengue Grave (consulta 1.4).
3. **Propriedade simétrica (`owl:SymmetricProperty`):** `:interageCom`. Declaramos `:losartana :interageCom :ibuprofeno`, e o reasoner infere o sentido oposto; o ASK 2.5 consulta justamente o sentido não declarado.
4. **Equivalência de classes (`owl:equivalentClass`):** `:Sintoma` ≡ `:ManifestacaoClinica`. `:mialgia` e `:anosmia` são declaradas apenas como `:ManifestacaoClinica`; o reasoner as classifica também como `:Sintoma` (consulta 1.7 e verificação 3.3). A `:anosmia` não é ligada a nenhuma doença, então a equivalência é a única causa dessa classificação.
5. **Propriedades funcionais (`owl:FunctionalProperty`):** `:causadaPor` (cada doença infecciosa tem um agente patogênico primário), `:codigoCID` (um código por doença) e `:idadePaciente` (uma idade por paciente). Para a propriedade de objeto `:causadaPor`, ligar duas URIs de agentes à mesma doença faz o reasoner inferir que são o mesmo indivíduo (`owl:sameAs`), como mostra a verificação 3.1. Nas propriedades de dados, a unicidade é uma declaração de modelagem: o reasoner OWL-RL não sinaliza erro quando valores literais distintos são atribuídos, então ela deve ser garantida na entrada dos dados.
6. **Disjunção de classes (`owl:disjointWith`):** entre `:DoencaViral` e `:DoencaBacteriana`, `:DoencaInfecciosa` e `:DoencaCronica`, `:Medicamento` e `:TerapiaSuporte`, e entre `:Doenca` e `:Tratamento`/`:Sintoma`. Um indivíduo classificado em duas classes disjuntas é sinalizado como inconsistente (verificação 3.2).

> Em OWL, `FunctionalProperty` e `disjointWith` não impedem a inserção de dados, como uma restrição de banco relacional faria (isso seria papel do SHACL). Eles fazem o reasoner **inferir** ou **detectar** consequências lógicas.

---

## 4. Base de dados (A-Box — 40 instâncias)

- **6 agentes patogênicos:** vírus da Dengue, Influenza A (H1N1), SARS-CoV-2, bacilo de Koch, meningococo e pneumococo.
- **10 doenças:**
  - 5 virais: Dengue Clássica, Dengue com Sinais de Alarme, Dengue Grave, Influenza A e COVID-19.
  - 3 bacterianas: Tuberculose, Pneumonia Bacteriana e Meningite Meningocócica.
  - 2 crônicas: Hipertensão e Asma Grave.
- **8 sintomas e manifestações clínicas:** Febre Alta, Cefaleia, Mialgia, Anosmia, Dispneia, Tosse Seca, Hemorragia e Rigidez na Nuca.
- **10 tratamentos:**
  - 8 medicamentos: Paracetamol, Dipirona, Oseltamivir, Amoxicilina, Rifampicina, Ceftriaxona, Losartana e Ibuprofeno.
  - 2 terapias de suporte: Hidratação Venosa e Oxigenoterapia.
- **6 pessoas:** 2 médicos e 4 pacientes, com idades e quadros clínicos diferentes.

Observação sobre o CID: a Dengue Clássica e a Dengue com Sinais de Alarme usam `A90` (no SUS, `A91` é reservado à forma hemorrágica/grave, usada na Dengue Grave).

---

## 5. Consultas e operações implementadas

O script `consultas_doencas.py` executa as consultas sobre o grafo expandido pelo reasoner. Cada consulta imprime seu número, seu propósito e, na Parte 1, o resultado esperado. A saída é ordenada, portanto idêntica a cada execução.

### Parte 1: navegação com `g.triples()` (8 consultas)

1. **`(sujeito, predicado, None)`:** sintomas da Dengue Clássica.
2. **`(None, predicado, objeto)`:** doenças que apresentam Dispneia.
3. **`(sujeito, predicado inferido, None)`:** de quais doenças a Febre Alta é sintoma, usando `:sintomaDe` (inferido via `owl:inverseOf`).
4. **`(None, predicado transitivo, None)`:** cadeias de evolução clínica com `:evoluiPara` (inclui o salto inferido por transitividade).
5. **`(sujeito, None, None)`:** todas as afirmações sobre a Rifampicina, incluindo a interação simétrica e a indicação inversa.
6. **`(None, rdf:type, :Doenca)`:** todas as doenças, independentemente do nível da hierarquia (inferência por `rdfs:subClassOf`).
7. **`(None, rdf:type, :Sintoma)`:** os 8 sintomas, incluindo Mialgia e Anosmia, que só são `:Sintoma` por `owl:equivalentClass`.
8. **`(None, None, objeto)`:** tudo que aponta para o Vírus da Dengue e por qual propriedade.

### Parte 2: consultas e atualizações SPARQL (9 operações)

1. **`SELECT` + `FILTER` + `ORDER BY`:** doenças com letalidade acima de 2,0%, da mais letal para a menos letal.
2. **`SELECT` + `VALUES` + `COUNT` + `GROUP BY` + `HAVING` (triagem estilo MYCIN):** dado um paciente com Febre Alta, Cefaleia e Mialgia, ranqueia as doenças pelo número de sintomas compatíveis, depois pela cobertura do quadro da doença e, em empate, pela maior letalidade.
3. **`SELECT` + `OPTIONAL` + `BIND`:** lista doenças e seus agentes patogênicos (opcionais, já que doenças crônicas não têm) e calcula o nível de risco com `IF`.
4. **`SELECT` com múltiplos joins e `GROUP_CONCAT`:** prontuário integrado com paciente, idade, doença, médico e tratamentos.
5. **`ASK`:** verifica se existe interação medicamentosa entre Ibuprofeno e Losartana.
6. **`CONSTRUCT`:** deriva um novo grafo ligando cada paciente diretamente aos tratamentos da sua doença (`:prescricaoSugerida`).
7. **`UPDATE` (`INSERT...WHERE`):** classifica como `:PacienteTriagemCritica` os pacientes com doença de letalidade ≥ 5,0% (mostra antes e depois).
8. **`UPDATE` (`DELETE DATA`):** revisão de protocolo clínico, removendo a indicação do Paracetamol para a Dengue Clássica. Remove 2 triplas: a original e a inversa já materializada pelo reasoner (mostra antes e depois).
9. **`UPDATE` (`DELETE...INSERT...WHERE`):** transição de estado, mudando o `:statusClinico` de "Em Triagem" para "Em Tratamento Supervisionado" nos pacientes que já têm médico atribuído (mostra antes e depois).

### Parte 3: verificação das construções OWL (3 testes)

Cada teste usa uma cópia nova do `.ttl` com uma tripla extra, sem alterar o grafo principal:

1. **`owl:FunctionalProperty`:** liga a Dengue Clássica a um segundo agente em `:causadaPor`; o reasoner infere `owl:sameAs` entre os dois (esperado: `True`).
2. **`owl:disjointWith`:** classifica um indivíduo como `:DoencaViral` e `:DoencaBacteriana` ao mesmo tempo (esperado: 0 inconsistências na base original e 1 ou mais no grafo de teste).
3. **`owl:equivalentClass`:** confirma que `:anosmia` virou `:Sintoma` sem passar por `:apresentaSintoma` (esperado: `True` e `True`).

---

## 6. Como executar

### Pré-requisitos

É necessário ter o Python 3 instalado. Instale as bibliotecas:

```bash
pip install rdflib owlrl
```

### Execução

Com `ontologia_doencas.ttl` e `consultas_doencas.py` na mesma pasta, execute:

```bash
python consultas_doencas.py
```

As primeiras linhas da saída devem ser:

```text
Total de triplas explícitas carregadas do arquivo .ttl: 325
Total de triplas após fechamento dedutivo (OWLRL_Semantics): 1017
Novas triplas inferidas automaticamente pelo Reasoner: 692
Inconsistências lógicas detectadas pelo Reasoner na base: 0
```
