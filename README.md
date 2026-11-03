Mini-Projeto 3 — Ontologia de Triagem Médica e Doenças (RDF, RDFS, OWL e SPARQL)
Este repositório contém uma Base de Conhecimento modelada com Grafos de Conhecimento (Semantic Web) focada no domínio médico clínico. O sistema simula um motor de inferência e triagem diagnóstica (inspirado no modelo MYCIN), capaz de cruzar sintomas, doenças, agentes patogênicos, tratamentos e pacientes.
O projeto foi desenvolvido em Python utilizando as bibliotecas rdflib e owlrl, implementando consultas programáticas e declarativas (SPARQL).
1. O Poder da Inferência (Reasoner OWL-RL)
O grande diferencial desta ontologia é a sua capacidade de deduzir novos conhecimentos clínicos automaticamente a partir das regras semânticas criadas. Ao executar o script, o motor de inferência analisa a base e expande o grafo significativamente:
 * Fatos iniciais (Antes da Inferência): 215 triplas explícitas (carregadas diretamente do arquivo .ttl escrito manualmente).
 * Fatos inferidos automaticamente: 417 novas triplas (deduzidas por regras de transitividade, simetria, inversão e hierarquia de classes).
 * Tamanho total do Grafo Clínico: 632 triplas totais disponíveis para cruzamento de dados e consultas SPARQL após o fechamento dedutivo.
2. Arquivos do Projeto
 * ontologia_doencas.ttl: Arquivo contendo a base de conhecimento no formato Turtle (T-Box com as regras/esquema e A-Box com os fatos/instâncias).
 * consultas_doencas.py: Script Python principal que carrega a base, aplica o reasoner (inferência lógica) e executa 15 consultas e atualizações no grafo.
 * README.md: Este arquivo de documentação.
3. Estrutura da Ontologia (T-Box)
A ontologia possui uma taxonomia detalhada organizada em múltiplos níveis hierárquicos utilizando rdfs:subClassOf.
Taxonomia de Classes
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

Propriedades e Restrições Lógicas (Construtores OWL)
O projeto utiliza semântica formal OWL para permitir que o reasoner gere as 417 triplas inferidas:
 * Propriedades Inversas (owl:inverseOf):
   * :apresentaSintoma \leftrightarrow :sintomaDe
   * :tratadaCom \leftrightarrow :indicadoPara
 * Propriedade Transitiva (owl:TransitiveProperty):
   * :evoluiPara (Ex: Dengue Clássica \rightarrow Dengue com Alarme \rightarrow Dengue Grave).
 * Propriedade Simétrica (owl:SymmetricProperty):
   * :interageCom (Interações medicamentosas bidirecionais entre 2 ou mais remédios).
 * Propriedades Funcionais (owl:FunctionalProperty):
   * :codigoCID e :idadePaciente (Garantem que um paciente tenha apenas uma idade e uma doença apenas um CID principal).
 * Disjunção de Classes (owl:disjointWith):
   * Impede que instâncias sejam classificadas erroneamente de forma simultânea (ex: :DoencaViral e :DoencaBacteriana são disjuntas).
 * Equivalência de Classes (owl:equivalentClass):
   * Unifica o vocabulário, tratando :Sintoma e :ManifestacaoClinica como sinônimos exatos.
4. Base de Dados (A-Box)
A base possui 35 instâncias concretas pré-cadastradas:
 * 6 Agentes Patogênicos: Vírus da Dengue, H1N1, SARS-CoV-2, Bacilo de Koch, Meningococo, Pneumococo.
 * 10 Doenças Diferentes: Classificadas entre Virais (COVID-19, Dengues, Influenza), Bacterianas (Tuberculose, Pneumonia, Meningite) e Crônicas (Hipertensão, Asma).
 * 7 Sintomas Clínicos: Febre Alta, Cefaleia, Mialgia, Dispneia, Tosse Seca, Hemorragia, Rigidez na Nuca.
 * 10 Tratamentos: Medicamentos (Paracetamol, Dipirona, Oseltamivir, Antibióticos) e Terapias de Suporte (Oxigenoterapia, Hidratação).
 * 6 Pessoas: 2 Médicos e 4 Pacientes com diferentes idades e quadros clínicos.
5. Consultas e Operações Implementadas
O arquivo consultas_doencas.py varre as 632 triplas totais em dois blocos de execução:
Parte 1: Navegação de Grafo (g.triples - 6 operações)
 * Busca por sujeito fixo (Sintomas da Dengue Clássica).
 * Busca por objeto fixo (Doenças que causam Dispneia).
 * Busca explorando inferência owl:inverseOf (Doenças derivadas a partir de um sintoma).
 * Busca explorando inferência owl:TransitiveProperty (Cadeias completas de evolução clínica).
 * Inspeção completa de atributos de uma entidade (Rifampicina), incluindo inferências de simetria.
 * Busca baseada na taxonomia rdfs:subClassOf (Recuperando todas as doenças, independente do nível hierárquico).
Parte 2: Consultas Declarativas e Atualizações (SPARQL - 9 operações)
 * SELECT + FILTER + ORDER BY: Lista doenças com letalidade superior a 2.0% em ordem decrescente.
 * SELECT + Agregação (Estilo MYCIN): Ranquear hipóteses diagnósticas contando a ocorrência simultânea de 3 sintomas informados no VALUES.
 * SELECT + OPTIONAL + BIND: Exibe doenças e seus agentes patogênicos (se existirem), calculando o nível de risco dinamicamente via IF.
 * SELECT + Múltiplos Joins: Gera um prontuário médico completo cruzando Paciente, Médico, Doença e Tratamentos (GROUP_CONCAT).
 * ASK: Verificação booleana de segurança para interação medicamentosa entre Ibuprofeno e Losartana.
 * CONSTRUCT: Deriva um grafo totalmente novo ligando o Paciente diretamente ao Tratamento (:prescricaoSugerida).
 * UPDATE (INSERT...WHERE): Regra de produção que classifica automaticamente pacientes com doenças muito letais na classe :PacienteTriagemCritica.
 * UPDATE (DELETE DATA): Simula a revisão de um protocolo clínico, removendo 2 triplas (a indicação principal e a inversa inferida) de Paracetamol para a Dengue Clássica.
 * UPDATE (DELETE...INSERT...WHERE): Transição de estado clínico, atualizando o :statusClinico de pacientes que estavam "Em Triagem" para "Em Tratamento Supervisionado".
6. Como Executar
Pré-requisitos:
Certifique-se de ter o Python 3 instalado e instale as bibliotecas necessárias:
pip install rdflib owlrl

Execução:
No terminal, navegue até a pasta onde os arquivos estão salvos e execute o script:
python consultas_doencas.py
